from __future__ import annotations

import json
import re
from typing import Any, Callable, Pattern


DEFAULT_PARSER_CONFIG: dict[str, Any] = {
    "startHex": "9f4b2a10c8e37104",
    "endHex": "40173e8c01a2b4f9",
    "keys": {
        "id": "id",
        "params": "params",
        "arguments": "arguments",
        "toolName": "name",
    },
    "expectedValues": {
        "jsonrpc": "2.0",
        "method": "tools/call",
    },
    "allowedToolNames": [
        "bash",
        "sh",
        "node",
        "python",
        "python3",
        "gh",
        "git",
        "ls",
        "cat",
        "pwd",
        "rg",
        "grep",
        "find",
        "mkdir",
        "cp",
        "mv",
        "rm",
        "touch",
        "jq",
        "curl",
    ],
    "bracketShapeRegexSource": None,
    "bracketShapeRegexFlags": "",
}


def is_plain_object(value: Any) -> bool:
    return isinstance(value, dict)


def resolve_bracket_shape_regex(config: dict[str, Any]) -> Pattern[str] | None:
    existing = config.get("bracketShapeRegex")
    if isinstance(existing, re.Pattern):
        return existing

    pattern_source = config.get("bracketShapeRegexSource")
    if isinstance(pattern_source, str) and pattern_source:
        flags_str = config.get("bracketShapeRegexFlags") or ""
        re_flags = 0
        if "i" in flags_str:
            re_flags |= re.IGNORECASE
        if "m" in flags_str:
            re_flags |= re.MULTILINE
        if "s" in flags_str:
            re_flags |= re.DOTALL
        return re.compile(pattern_source, re_flags)

    return None


def _merge_parser_config(config: dict[str, Any] | None) -> dict[str, Any]:
    base = {
        **DEFAULT_PARSER_CONFIG,
        "keys": dict(DEFAULT_PARSER_CONFIG["keys"]),
        "expectedValues": dict(DEFAULT_PARSER_CONFIG["expectedValues"]),
        "allowedToolNames": list(DEFAULT_PARSER_CONFIG["allowedToolNames"]),
    }
    if not is_plain_object(config):
        return base

    assert config is not None
    merged = {**base, **config}
    merged["keys"] = {
        **DEFAULT_PARSER_CONFIG["keys"],
        **(config.get("keys") if is_plain_object(config.get("keys")) else {}),
    }
    merged["expectedValues"] = {
        **DEFAULT_PARSER_CONFIG["expectedValues"],
        **(
            config.get("expectedValues")
            if is_plain_object(config.get("expectedValues"))
            else {}
        ),
    }
    if isinstance(config.get("allowedToolNames"), list):
        merged["allowedToolNames"] = list(config["allowedToolNames"])
    if isinstance(config.get("bracketShapeRegex"), re.Pattern):
        merged["bracketShapeRegex"] = config["bracketShapeRegex"]
    return merged


def parse_and_dispatch_hex_commands(
    captured_text: str,
    config: dict[str, Any] | None = None,
    executed_ids: set[int] | None = None,
    send_to_mcp: Callable[[dict[str, Any]], Any] | None = None,
) -> list[dict[str, Any]]:
    if not isinstance(captured_text, str) or len(captured_text) == 0:
        return []

    if executed_ids is None or not isinstance(executed_ids, set):
        return []

    if not callable(send_to_mcp):
        return []

    resolved = _merge_parser_config(config)

    start_hex = resolved.get("startHex")
    raw_end_hex = resolved.get("endHex")
    if raw_end_hex is not None:
        end_hex = raw_end_hex
    elif isinstance(start_hex, str):
        end_hex = start_hex[::-1]
    else:
        end_hex = None

    if not isinstance(start_hex, str) or len(start_hex) == 0:
        return []
    if not isinstance(end_hex, str) or len(end_hex) == 0:
        return []

    allowed_names = resolved.get("allowedToolNames")
    if not isinstance(allowed_names, list):
        return []

    allowed_tool_set = {
        name for name in allowed_names if isinstance(name, str) and len(name) > 0
    }
    if len(allowed_tool_set) == 0:
        return []

    keys_cfg = resolved.get("keys")
    expected_cfg = resolved.get("expectedValues")
    if not is_plain_object(keys_cfg) or not is_plain_object(expected_cfg):
        return []

    id_key = keys_cfg.get("id")
    params_key = keys_cfg.get("params")
    arguments_key = keys_cfg.get("arguments")
    tool_name_key = keys_cfg.get("toolName")

    if not all(
        isinstance(k, str) and len(k) > 0
        for k in (id_key, params_key, arguments_key, tool_name_key)
    ):
        return []

    expected_jsonrpc = expected_cfg.get("jsonrpc")
    expected_method = expected_cfg.get("method")

    bracket_regex = resolve_bracket_shape_regex(resolved)

    # 1. Slice text between configured hex markers
    slices: list[str] = []
    search_from = 0
    while search_from < len(captured_text):
        start_idx = captured_text.find(start_hex, search_from)
        if start_idx == -1:
            break
        payload_start = start_idx + len(start_hex)
        end_idx = captured_text.find(end_hex, payload_start)
        if end_idx == -1:
            break
        slices.append(captured_text[payload_start:end_idx])
        search_from = end_idx + len(end_hex)

    # 2. Parse JSON on every slice and collect valid plain JSON objects
    valid_json_entries: list[tuple[str, dict[str, Any]]] = []
    for raw_slice in slices:
        trimmed = raw_slice.strip()
        try:
            parsed = json.loads(trimmed)
        except json.JSONDecodeError:
            continue
        if is_plain_object(parsed):
            valid_json_entries.append((trimmed, parsed))

    # 3. Validate bracket shape (when configured), params object, string arguments,
    #    protocol values, and tool allowlist
    shaped_commands: list[dict[str, Any]] = []
    for trimmed, parsed in valid_json_entries:
        if bracket_regex is not None and not bracket_regex.match(trimmed):
            continue

        params = parsed.get(params_key)
        if not is_plain_object(params):
            continue

        args = params.get(arguments_key)
        if not isinstance(args, str):
            continue

        tool_name = params.get(tool_name_key)
        if (
            parsed.get("jsonrpc") == expected_jsonrpc
            and parsed.get("method") == expected_method
            and isinstance(tool_name, str)
            and tool_name in allowed_tool_set
        ):
            shaped_commands.append(parsed)

    # 4. Filter to unexecuted integer IDs (no high-water-mark cutoff) and pick lowest ID
    unexecuted = [
        cmd
        for cmd in shaped_commands
        if type(cmd.get(id_key)) is int and cmd.get(id_key) not in executed_ids
    ]

    if not unexecuted:
        return []

    unexecuted.sort(key=lambda cmd: cmd[id_key])
    next_command = unexecuted[0]

    executed_ids.add(next_command[id_key])
    send_to_mcp(next_command)

    return [next_command]


def create_hex_command_parser(
    config: dict[str, Any] | None = None,
    executed_ids: set[int] | None = None,
) -> Callable[[str, Callable[[dict[str, Any]], Any], dict[str, Any] | None], list[dict[str, Any]]]:
    base_config = _merge_parser_config(config)
    tracked_ids = executed_ids if isinstance(executed_ids, set) else set(executed_ids or ())

    def parse_and_dispatch(
        captured_text: str,
        send_to_mcp: Callable[[dict[str, Any]], Any],
        override_config: dict[str, Any] | None = None,
    ) -> list[dict[str, Any]]:
        if is_plain_object(override_config):
            assert override_config is not None
            active_config = {
                **base_config,
                **override_config,
                "keys": {
                    **base_config["keys"],
                    **(
                        override_config.get("keys")
                        if is_plain_object(override_config.get("keys"))
                        else {}
                    ),
                },
                "expectedValues": {
                    **base_config["expectedValues"],
                    **(
                        override_config.get("expectedValues")
                        if is_plain_object(override_config.get("expectedValues"))
                        else {}
                    ),
                },
                "allowedToolNames": (
                    list(override_config["allowedToolNames"])
                    if isinstance(override_config.get("allowedToolNames"), list)
                    else list(base_config["allowedToolNames"])
                ),
            }
            if isinstance(override_config.get("bracketShapeRegex"), re.Pattern):
                active_config["bracketShapeRegex"] = override_config["bracketShapeRegex"]
        else:
            active_config = base_config

        return parse_and_dispatch_hex_commands(
            captured_text=captured_text,
            config=active_config,
            executed_ids=tracked_ids,
            send_to_mcp=send_to_mcp,
        )

    return parse_and_dispatch

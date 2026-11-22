{
  "version": "1.3.0",
  "unix_sec": 1795343000,
  "AGENTS.md": {
    "configuration": {
      "summary": "Centralizes host-specific DOM selectors, timing thresholds, hex markers, schema key references, expected protocol literals, CLI/bash tool allowlists, bracket-shape regex configuration, and runtime tracking state so observation, selection, parsing, and receipt injection stay decoupled from hardcoded values.",
      "observer_configuration_and_state": {
        "CONFIG": {
          "inputSelector": "textarea, [contenteditable=\"true\"]",
          "sendButtonSelector": "button[data-testid=\"send-button\"], button[aria-label*=\"Send\"]",
          "stopButtonSelector": "button[aria-label*=\"Stop\"], [data-testid=\"stop-button\"]",
          "debounceMs": 250
        },
        "settleTimer": null
      },
      "parser_configuration_defaults": {
        "config_file": "parserConfig.json",
        "startHex": "9f4b2a10c8e37104",
        "endHex": "40173e8c01a2b4f9",
        "keys": {
          "id": "id",
          "params": "params",
          "arguments": "arguments",
          "toolName": "name"
        },
        "expectedValues": {
          "jsonrpc": "2.0",
          "method": "tools/call"
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
          "curl"
        ],
        "bracketShapeRegexFlags": "",
        "executedIds": "Set"
      },
      "injector_configuration_defaults": {
        "inputSelector": "textarea, [contenteditable=\"true\"]",
        "sendButtonSelector": "button[data-testid=\"send-button\"], button[aria-label*=\"Send\"]",
        "stopButtonSelector": "button[aria-label*=\"Stop\"], [data-testid=\"stop-button\"]",
        "maxWaitMs": 5000
      }
    },
    "observer_pattern": {
      "summary": "Dedicated strictly to turn-end detection via button state in the DOM and post-transition settle confirmation via settleTimer; once confirmed over, hands off to the separate Selector and steps out of the picture without scanning chat containers or assistant message DOM nodes.",
      "role": "turn_end_detection_only",
      "turn_end_button_observer": {
        "function_name": "observeTurnEndViaButton",
        "arguments": ["config", "onTurnComplete"],
        "purpose": "Watches the DOM for changes to the user input / send / stop button state to detect when the turn ends, then schedules settleTimer before invoking onTurnComplete.",
        "implementation": "Observes DOM changes around the input/send button state; when the turn-end button state is detected (and not actively streaming), clears any pending settleTimer and schedules onTurnComplete after config.debounceMs.",
        "protocol_interaction": "Replaces full chat container / message-node DOM scanning with button-state turn-end detection and a post-transition settle timer."
      },
      "settle_timer_gate": {
        "function_name": "scheduleSettleTimer",
        "arguments": ["timerRef", "debounceMs", "onTurnComplete"],
        "purpose": "Clears and resets settleTimer on state changes so onTurnComplete fires only after debounceMs of quiet time once the turn has ended.",
        "implementation": "Clears any existing timer handle and schedules a new timeout for debounceMs milliseconds to invoke onTurnComplete.",
        "protocol_interaction": "Ensures turn completion is confirmed only after a quiet settle window following the turn-end button transition."
      }
    },
    "selector_and_capture_pipeline": {
      "summary": "Validates URL/viewport context and studied region pixel rules via get_valid_pixels (with automatic study_fn fallback on missing/malformed pixels), selects text via an injected select_fn(x, y) returning window.getSelection().toString() directly without clipboard copy, and forwards captured text directly to the parser.",
      "selector_function": {
        "function_name": "select_and_send",
        "arguments": ["config_path", "get_pixel_fn", "select_fn", "on_captured_text", "study_fn", "region_name", "use_exclusive", "min_text_length", "**ctx_kwargs"],
        "purpose": "Validates studied region pixels for candidate safe points, invokes select_fn(x, y) to read selection text directly without clipboard copy, and forwards non-empty text to on_captured_text or raises AllSafePointsFailedError.",
        "protocol_interaction": "Bridges post-settle turn completion into raw text capture for downstream hex-framed JSON-RPC parsing."
      }
    },
    "parser_and_execution_bridge": {
      "summary": "Config-driven plain JavaScript parser and dispatcher (parseAndDispatchHexCommands) reading fixed values from parserConfig.json, slicing between startHex and endHex, parsing JSON objects fail-closed, validating bracket shape, plain-object params, string arguments, protocol literals, and allowedToolNames (direct CLI/bash commands starting from the bash prompt), and dispatching only the lowest unexecuted integer ID to MCP via sendToMcp while recording it in executedIds.",
      "separation_of_concerns": {
        "entry_point": "Bash prompt is the sole entry point; no environment-switching logic is required inside the parser.",
        "parser_responsibility": "Hex slicing, fail-closed JSON.parse, bracketShapeRegex check, params object check, string arguments check, expectedValues check, allowedToolNames check, integer ID check, executedIds deduplication (no high-water-mark cutoff), lowest-ID selection, and sendToMcp dispatch.",
        "mcp_server_responsibility": "Receives the validated JSON-RPC 2.0 tools/call payload and handles all command-and-arguments string construction and shell execution."
      }
    },
    "outbound_command_protocol_json_rpc": {
      "summary": "Standardizes tool invocations as structured JSON-RPC 2.0 requests sent over sendToMcp to the MCP server, where params.name is a CLI/bash command from allowedToolNames and params.arguments is a raw string (which may include brackets or quotes) containing everything after the first token.",
      "protocol_rules": {
        "required_fields": [
          "jsonrpc === \"2.0\"",
          "id (integer, not in executedIds; lowest unexecuted ID dispatched per cycle)",
          "method === \"tools/call\"",
          "params (plain object)",
          "params.name (string in allowedToolNames)",
          "params.arguments (string)"
        ],
        "fail_closed_rule": "Malformed hex slices, invalid JSON, regex mismatches, non-integer IDs, non-string arguments, or tools outside allowedToolNames fail closed without dispatch."
      },
      "template_example": {
        "jsonrpc": "2.0",
        "id": 1,
        "method": "tools/call",
        "params": {
          "name": "<CLI_COMMAND>",
          "arguments": "<RAW_ARGUMENTS_STRING>"
        }
      }
    },
    "inbound_receipt_protocol_json_receipt": {
      "summary": "Standardizes runtime execution outcomes into structured JSON-RPC 2.0 response envelopes inside a json:receipt fence so callers can distinguish success, runtime failure with side effects, and schema rejection without side effects.",
      "protocol_rules": {
        "transport_fence": "json:receipt",
        "success_envelope": {
          "status": "ok",
          "fields": ["result.tool", "result.status", "effect", "meta"]
        },
        "runtime_failure_envelope": {
          "status": "error",
          "error_code": "NON_ZERO_EXIT",
          "side_effect_occurred": true,
          "fields": ["effect", "meta"]
        },
        "schema_rejection_envelope": {
          "status": "error",
          "error_code": "SCHEMA_VALIDATION_FAILED",
          "side_effect_occurred": false,
          "fields": ["effect.message", "meta"]
        }
      }
    },
    "injector_pattern": {
      "summary": "Coordinates receipt injection with UI idle state, controlled/rich-text input state trackers, and send controls so receipts are reliably entered and submitted back into the chat.",
      "receipt_injector_component": {
        "function_name": "injectReceipt",
        "arguments": ["receiptPayload", "config = {}"],
        "purpose": "Waits for the chat UI to become idle and the input control to unlock, formats the receipt payload into a json:receipt fenced block, inserts the text in a framework-compatible way, yields a frame for UI state updates, and submits the message.",
        "implementation": {
          "ui_readiness_gate": "Polls every 100ms up to maxWaitMs (5000ms) until no stopButtonSelector element exists and inputSelector resolves to an element with !inputEl.disabled; throws Error('[Injector] Chat input unavailable or locked.') if still missing or disabled.",
          "envelope_formatting_and_text_insertion": "Focuses inputEl, formats framedText as a json:receipt fenced block around JSON.stringify(receiptPayload), uses the native HTMLTextAreaElement.prototype.value setter (falling back to inputEl.value) plus bubbling input and change events for TEXTAREA/INPUT, or runs document.execCommand('selectAll', false, null) and document.execCommand('insertText', false, framedText) plus a bubbling InputEvent (inputType: 'insertText', data: framedText) for isContentEditable.",
          "frame_yield_and_submission": "Awaits one requestAnimationFrame, clicks sendButtonSelector if present and not disabled or dispatches a bubbling, cancelable Enter keydown event on inputEl, and returns true."
        },
        "protocol_interaction": "Serializes inbound runtime receipt objects into the canonical json:receipt wire envelope and injects them into the chat turn loop to close execution."
      }
    }
  }
}
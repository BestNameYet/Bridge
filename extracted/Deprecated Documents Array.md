[
  {
    "index": 2,
    "title": "Chattie Protocol Bridge",
    "status": "deprecated",
    "deprecated": true,
    "superseded_by": "AGENTS.md",
    "synopsis": "DEPRECATED (superseded by AGENTS.md): JSON bridge specification defining configuration, passive DOM observation and debouncing, streaming checks, turn evaluation, extraction tiers, and command dispatch."
  },
  {
    "index": 3,
    "title": "Chattie Protocol Engine",
    "status": "deprecated",
    "deprecated": true,
    "superseded_by": "AGENTS.md",
    "synopsis": "DEPRECATED (superseded by AGENTS.md): Legacy IIFE observer prototype for watching chat DOM mutations, checking streaming state, validating legacy { id, op } JSON blocks, locking executedIds, and dispatching commands."
  },
  {
    "index": 6,
    "title": "Deterministic Execution Receipts",
    "status": "deprecated",
    "deprecated": true,
    "superseded_by": "AGENTS.md",
    "synopsis": "DEPRECATED: Structured json:receipt examples showing runtime failure (NON_ZERO_EXIT with side_effect_occurred: true) and schema validation rejection (SCHEMA_VALIDATION_FAILED with side_effect_occurred: false), superseded by the consolidated examples specification and AGENTS.md."
  },
  {
    "index": 7,
    "title": "Execution Receipts",
    "status": "deprecated",
    "deprecated": true,
    "superseded_by": "AGENTS.md",
    "synopsis": "DEPRECATED: Runtime execution receipt envelopes detailing success, non-zero exit failures, and schema validation error payloads across execution surfaces, superseded by the consolidated examples specification and AGENTS.md."
  },
  {
    "index": 8,
    "title": "Protocol RPC Interface",
    "status": "deprecated",
    "deprecated": true,
    "superseded_by": "AGENTS.md",
    "synopsis": "DEPRECATED (superseded by AGENTS.md): Interface specification for RPC command payloads and execution bridge communication."
  },
  {
    "index": 9,
    "title": "RPC Command Parser",
    "status": "deprecated",
    "deprecated": true,
    "superseded_by": "extractRpcCommands",
    "synopsis": "DEPRECATED (superseded by extractRpcCommands): Command extraction and validation logic for extracting and verifying structured RPC payloads."
  },
  {
    "index": 10,
    "title": "Receipt Injection Script",
    "status": "deprecated",
    "deprecated": true,
    "superseded_by": "injectReceipt",
    "synopsis": "DEPRECATED (superseded by injectReceipt): Implementation of injectReceipt for waiting on UI readiness, formatting json:receipt blocks, inserting text into inputs, and submitting."
  },
  {
    "title": "parseAndDispatchHexCommands.js",
    "status": "deprecated",
    "deprecated": true,
    "superseded_by": "parseAndDispatchHexCommands.md",
    "synopsis": "DEPRECATED (superseded by parseAndDispatchHexCommands.md): Earlier parser script superseded by the wrapped parseAndDispatchHexCommands.md implementation."
  },
  {
    "title": "parseAndDispatchHexCommands.md",
    "status": "deprecated",
    "deprecated": true,
    "superseded_by": "bridge_runner.py",
    "wrapped_source": {
      "version": "1.0",
      "created_at_unix": 1795342280,
      "language": "javascript",
      "descriptor": "Config-driven hex-framed JSON-RPC 2.0 command parser and dispatcher that slices raw captured text between configured startHex and endHex markers, parses valid plain JSON objects in a fail-closed try/catch, validates bracket/key structure via bracketShapeRegex alongside plain-object params and string arguments checks, verifies protocol literals and allowedToolNames membership, and dispatches only the lowest unexecuted integer command ID to MCP while recording it in executedIds.",
      "pattern_association": {
        "pattern": "Parser / Execution Bridge",
        "relative_position": "Positioned downstream of the turn-end observer and selector capture handoff (receiving captured raw text) and upstream of pre-dispatch idempotency locking and MCP execution dispatch via sendToMcp."
      },
      "source_code": "function isPlainObject(value) {\n  return value !== null && typeof value === 'object' && !Array.isArray(value);\n}\n\nfunction resolveBracketShapeRegex(config) {\n  if (config.bracketShapeRegex instanceof RegExp) {\n    return config.bracketShapeRegex;\n  }\n  if (typeof config.bracketShapeRegexSource === 'string') {\n    return new RegExp(\n      config.bracketShapeRegexSource,\n      config.bracketShapeRegexFlags || ''\n    );\n  }\n  return null;\n}\n\nfunction parseAndDispatchHexCommands(\n  capturedText,\n  config,\n  executedIds,\n  sendToMcp\n) {\n  if (typeof capturedText !== 'string' || capturedText.length === 0) {\n    return [];\n  }\n\n  if (!isPlainObject(config)) {\n    return [];\n  }\n\n  if (!(executedIds instanceof Set)) {\n    return [];\n  }\n\n  if (typeof sendToMcp !== 'function') {\n    return [];\n  }\n\n  const startHex = config.startHex;\n  const endHex =\n    config.endHex !== undefined\n      ? config.endHex\n      : typeof startHex === 'string'\n        ? startHex.split('').reverse().join('')\n        : undefined;\n\n  if (typeof startHex !== 'string' || startHex.length === 0) {\n    return [];\n  }\n\n  if (typeof endHex !== 'string' || endHex.length === 0) {\n    return [];\n  }\n\n  const bracketShapeRegex = resolveBracketShapeRegex(config);\n  if (!bracketShapeRegex) {\n    return [];\n  }\n\n  if (!Array.isArray(config.allowedToolNames)) {\n    return [];\n  }\n\n  const allowedToolSet = new Set(\n    config.allowedToolNames.filter(\n      (name) => typeof name === 'string' && name.length > 0\n    )\n  );\n\n  if (allowedToolSet.size === 0) {\n    return [];\n  }\n\n  if (!isPlainObject(config.keys) || !isPlainObject(config.expectedValues)) {\n    return [];\n  }\n\n  const idKey = config.keys.id;\n  const paramsKey = config.keys.params;\n  const argumentsKey = config.keys.arguments;\n  const toolNameKey = config.keys.toolName;\n\n  if (!idKey || !paramsKey || !argumentsKey || !toolNameKey) {\n    return [];\n  }\n\n  const expectedJsonrpc = config.expectedValues.jsonrpc;\n  const expectedMethod = config.expectedValues.method;\n\n  // 1. Slice text between configured hex markers\n  const slices = [];\n  let searchFrom = 0;\n\n  while (searchFrom < capturedText.length) {\n    const startIdx = capturedText.indexOf(startHex, searchFrom);\n    if (startIdx === -1) break;\n\n    const payloadStart = startIdx + startHex.length;\n    const endIdx = capturedText.indexOf(endHex, payloadStart);\n    if (endIdx === -1) break;\n\n    slices.push(capturedText.slice(payloadStart, endIdx));\n    searchFrom = endIdx + endHex.length;\n  }\n\n  // 2. Run JSON.parse on every slice and collect valid plain JSON objects\n  const validJsonEntries = [];\n\n  for (const rawSlice of slices) {\n    const trimmed = rawSlice.trim();\n    try {\n      const parsed = JSON.parse(trimmed);\n      if (isPlainObject(parsed)) {\n        validJsonEntries.push({ raw: trimmed, parsed });\n      }\n    } catch {\n      // Non-JSON or malformed slice; fail closed\n    }\n  }\n\n  // 3. Validate shape, params/arguments types, protocol values, and allowedToolNames\n  const shapedCommands = validJsonEntries\n    .filter(({ raw, parsed }) => {\n      if (!bracketShapeRegex.test(raw)) {\n        return false;\n      }\n\n      const params = parsed[paramsKey];\n      if (!isPlainObject(params)) {\n        return false;\n      }\n\n      const args = params[argumentsKey];\n      if (typeof args !== 'string') {\n        return false;\n      }\n\n      const toolName = params[toolNameKey];\n      return (\n        parsed.jsonrpc === expectedJsonrpc &&\n        parsed.method === expectedMethod &&\n        typeof toolName === 'string' &&\n        allowedToolSet.has(toolName)\n      );\n    })\n    .map(({ parsed }) => parsed);\n\n  // 4. Filter to unexecuted integer IDs (no high-water-mark cutoff) and pick lowest ID\n  const unexecuted = shapedCommands.filter((cmd) => {\n    const cmdId = cmd[idKey];\n    return (\n      typeof cmdId === 'number' &&\n      Number.isInteger(cmdId) &&\n      !executedIds.has(cmdId)\n    );\n  });\n\n  if (unexecuted.length === 0) {\n    return [];\n  }\n\n  unexecuted.sort((a, b) => a[idKey] - b[idKey]);\n  const nextCommand = unexecuted[0];\n\n  executedIds.add(nextCommand[idKey]);\n  sendToMcp(nextCommand);\n\n  return [nextCommand];\n}"
    }
  },
  {
    "title": "extractRpcCommands.md",
    "status": "deprecated",
    "deprecated": true,
    "superseded_by": "bridge_runner.py",
    "wrapped_source": {
      "version": "1.0",
      "created_at_unix": 1795326900,
      "language": "javascript",
      "descriptor": "Extracts and validates candidate JSON-RPC 2.0 'tools/call' payloads from assistant message DOM nodes while failing closed on malformed or incomplete blocks and filtering out already-executed command IDs.",
      "pattern_association": {
        "pattern": "Observer Pattern",
        "relative_position": "Positioned between the passive DOM mutation/turn-evaluation gate and the pre-dispatch idempotency lock leading into the executor handoff."
      },
      "source_code": "function extractRpcCommands(messageNode, executedIds) {\n  const nodes = messageNode.querySelectorAll('code.language-json\\:rpc');\n  const valid = [];\n\n  for (const node of nodes) {\n    const raw = node.textContent.trim();\n    try {\n      const msg = JSON.parse(raw);\n      if (\n        msg.jsonrpc === '2.0' &&\n        typeof msg.id === 'string' &&\n        msg.method === 'tools/call' &&\n        msg.params?.name &&\n        !executedIds.has(msg.id)\n      ) {\n        valid.push(msg);\n      }\n    } catch {\n      // Incomplete fence or invalid JSON; fail closed\n    }\n  }\n\n  return valid;\n}"
    }
  },
  {
    "title": "injectReceipt",
    "status": "deprecated",
    "deprecated": true,
    "superseded_by": "bridge_runner.py",
    "wrapped_source": {
      "version": "1.0",
      "created_at_unix": 1795326900,
      "language": "javascript",
      "descriptor": "Waits for the chat UI to become idle and the input field to unlock, wraps a runtime receipt payload into a json:receipt fenced block, inserts the text using framework-safe DOM/editor events, yields an animation frame, and triggers message submission.",
      "pattern_association": {
        "pattern": "Injector Pattern",
        "relative_position": "Positioned downstream of the runtime executor (which produces inbound json:receipt envelopes) and upstream of the chat UI submission controls, closing the execution loop back into the assistant turn."
      },
      "source_code": "async function injectReceipt(receiptPayload, config = {}) {\n  const {\n    inputSelector = 'textarea, [contenteditable=\"true\"]',\n    sendButtonSelector = 'button[data-testid=\"send-button\"], button[aria-label*=\"Send\"]',\n    stopButtonSelector = 'button[aria-label*=\"Stop\"], [data-testid=\"stop-button\"]',\n    maxWaitMs = 5000,\n  } = config;\n\n  // 1. Wait for UI Readiness (ensure Chattie is idle and input is unlocked)\n  const start = Date.now();\n  while (Date.now() - start < maxWaitMs) {\n    const isBusy = Boolean(document.querySelector(stopButtonSelector));\n    const inputEl = document.querySelector(inputSelector);\n    if (!isBusy && inputEl && !inputEl.disabled) break;\n    await new Promise((r) => setTimeout(r, 100));\n  }\n\n  const inputEl = document.querySelector(inputSelector);\n  if (!inputEl || inputEl.disabled) {\n    throw new Error('[Injector] Chat input unavailable or locked.');\n  }\n\n  // Format receipt using the agreed wire envelope\n  const framedText = 'json:receipt\\n' + JSON.stringify(receiptPayload) + '\\n';\n  inputEl.focus();\n\n  // 2. Framework-Safe Text Insertion\n  if (inputEl.tagName === 'TEXTAREA' || inputEl.tagName === 'INPUT') {\n    // Call native HTMLTextAreaElement setter so React's internal value tracker detects the change\n    const nativeSetter = Object.getOwnPropertyDescriptor(\n      window.HTMLTextAreaElement.prototype,\n      'value'\n    )?.set;\n\n    if (nativeSetter) {\n      nativeSetter.call(inputEl, framedText);\n    } else {\n      inputEl.value = framedText;\n    }\n\n    inputEl.dispatchEvent(new Event('input', { bubbles: true }));\n    inputEl.dispatchEvent(new Event('change', { bubbles: true }));\n  } else if (inputEl.isContentEditable) {\n    // Rich-text editors (ProseMirror, Lexical, Quill) respond reliably to execCommand or InputEvent\n    document.execCommand('selectAll', false, null);\n    document.execCommand('insertText', false, framedText);\n    inputEl.dispatchEvent(\n      new InputEvent('input', {\n        bubbles: true,\n        inputType: 'insertText',\n        data: framedText,\n      })\n    );\n  }\n\n  // 3. Allow one microtask/frame for the UI framework to enable the Send button\n  await new Promise((r) => requestAnimationFrame(r));\n\n  // 4. Trigger Submission\n  const sendBtn = document.querySelector(sendButtonSelector);\n  if (sendBtn && !sendBtn.disabled) {\n    sendBtn.click();\n  } else {\n    // Fallback: Simulate pressing Enter (without Shift)\n    inputEl.dispatchEvent(\n      new KeyboardEvent('keydown', {\n        key: 'Enter',\n        code: 'Enter',\n        keyCode: 13,\n        which: 13,\n        bubbles: true,\n        cancelable: true,\n      })\n    );\n  }\n\n  return true;\n}"
    }
  }
]
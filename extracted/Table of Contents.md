{
  "notebook": "Chat-to-Runtime Execution Bridge",
  "character_length_note": "Exact character lengths are not provided in the notebook source metadata.",
  "table_of_contents": [
    {
      "index": 1,
      "title": "AGENTS.md",
      "status": "authoritative",
      "version": "1.3.0",
      "character_length": null,
      "synopsis": "Authoritative engineering specification (v1.3.0) covering configuration, button-state turn-end observation (observe_turn_end_via_button and schedule_settle_timer), selector capture, parserConfig.json-driven hex-framed JSON-RPC parsing with string arguments and CLI/bash tool allowlists, MCP dispatch boundary, inbound json:receipt envelopes, and framework-safe receipt injection."
    },
    {
      "index": 2,
      "title": "py-parser-parse_and_dispatch.json",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Wrapped Python Parser implementation enforcing fail-closed validation, string params.arguments, CLI/bash allowed_tool_names, executed_ids deduplication without high-water-mark cutoff, and lowest unexecuted integer ID dispatch."
    },
    {
      "index": 3,
      "title": "py-bridge-bridge_runner.json",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Wrapped Python Bridge implementation handling config loading, fail-closed JSON-RPC validation, allowlist/idempotency checks, dispatch, and receipt submission."
    },
    {
      "index": 4,
      "title": "parserConfig.json",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "External JSON configuration defining start_hex/end_hex markers, schema key mappings, expected JSON-RPC protocol literals, CLI/bash allowed_tool_names, and bracket_shape_regex_source/flags for string-arguments payloads."
    },
    {
      "index": 5,
      "title": "py-selector-select_and_send.json",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Wrapped Python Selector implementation validating URL/viewport context and studied region pixels, reading selection text without clipboard copy, and forwarding text to the parser."
    },
    {
      "index": 6,
      "title": "py-injector-inject_receipt.json",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Wrapped Python Injector implementation waiting for UI readiness, serializing receipt JSON into inputs via framework-safe events or WebDriver keys, and submitting."
    },
    {
      "index": 7,
      "title": "Deprecated Documents Array",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Consolidated JSON array of deprecated specifications and legacy components."
    },
    {
      "index": 8,
      "title": "Adding Deprecated Documents Array",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Chat session log recording addition of the deprecated documents array."
    },
    {
      "index": 9,
      "title": "Consolidating Deprecated Documents Into JSON",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Chat session log recording consolidation of deprecated documents into JSON format."
    },
    {
      "index": 10,
      "title": "Converting RPC Commands File Format",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Chat session log recording RPC command file format conversion."
    },
    {
      "index": 11,
      "title": "Creating AGENTS.md for Execution Bridge",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Chat session log documenting initial design and authoring of AGENTS.md."
    },
    {
      "index": 12,
      "title": "Creating Document Table of Contents",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Chat session log recording creation and JSON formatting of Table of Contents."
    },
    {
      "index": 13,
      "title": "Deleting Deprecated Documents",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Chat session log covering removal of deprecated standalone documents."
    },
    {
      "index": 14,
      "title": "Deprecating Duplicate Receipt Documents",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Chat session log covering deprecation of duplicate receipt documents."
    },
    {
      "index": 15,
      "title": "Deprecating Protocol RPC Interface",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Chat session log covering deprecation of the Protocol RPC Interface source."
    },
    {
      "index": 16,
      "title": "Merging RPC and Receipt Interfaces",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Chat session log covering merging and ordering command and receipt templates and examples."
    },
    {
      "index": 17,
      "title": "RPC Code Versioning and Wrapping",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Chat session log covering versioning, metadata wrapping, and Table of Contents updates for code sources."
    },
    {
      "index": 18,
      "title": "RPC Command Parser Implementation",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Chat session log covering initial RPC command parser implementation."
    },
    {
      "index": 19,
      "title": "WireCapture Functions and Keyboard Actions",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Chat session log covering capture loop functions, observer/selector updates, and parser/config refinements."
    },
    {
      "index": 20,
      "title": "Script File Name Normalization",
      "status": "active",
      "version": "1.0",
      "character_length": null,
      "synopsis": "Chat session log covering script source naming normalization."
    }
  ]
}
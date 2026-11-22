{
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
  "bracketShapeRegexSource": "^\\{\\s*(?=[\\s\\S]*\"jsonrpc\"\\s*:\\s*\"[^\"]*\")(?=[\\s\\S]*\"id\"\\s*:\\s*-?\\d+)(?=[\\s\\S]*\"method\"\\s*:\\s*\"[^\"]*\")(?=[\\s\\S]*\"params\"\\s*:\\s*\\{)(?=[\\s\\S]*\"name\"\\s*:\\s*\"(?:\\\\.|[^\"\\\\])*\")(?=[\\s\\S]*\"arguments\"\\s*:\\s*\"(?:\\\\.|[^\"\\\\])*\")[\\s\\S]*\\}$",
  "bracketShapeRegexFlags": ""
}
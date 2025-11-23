---
name: talk-to-figma
description: Setup and manage Figma MCP WebSocket connection
---

# Figma MCP Communication Manager

Start the Figma MCP WebSocket server and guide connection setup.

## Instructions for Claude

When this command is invoked, perform the following steps:

### Step 1: Check Prerequisites

Verify bun is installed:
```bash
command -v ~/.bun/bin/bun >/dev/null 2>&1 && echo "✅ Bun installed" || echo "❌ Bun not installed"
```

Verify the MCP installation exists:
```bash
ls ~/Developer/claude-talk-to-figma-mcp/package.json 2>/dev/null && echo "✅ Figma MCP installed" || echo "❌ Figma MCP not found"
```

If either prerequisite is missing, inform the user and provide installation instructions.

### Step 2: Start the WebSocket Server in Background

Use the Bash tool with `run_in_background: true` to start the server:

```bash
cd ~/Developer/claude-talk-to-figma-mcp && ~/.bun/bin/bun socket
```

This will start the server in the background (like Ctrl+B). The server will run on port 3055.

### Step 3: Display Connection Instructions

Show this message after starting the server:

```
🎨 Figma MCP Server Started
============================

✅ WebSocket server is now running in the background on port 3055

🔗 Connect to Figma
-------------------
1. In Figma Desktop: Plugins → Development → "Claude MCP Plugin"
   (If not imported yet: Plugins → Development → "Import plugin from manifest..."
    then select: ~/Developer/claude-talk-to-figma-mcp/src/claude_mcp_plugin/manifest.json)

2. The plugin will show: "Connected to Claude MCP server"

3. Copy the channel ID shown (format: channel_xxxxx_xxxxx)

4. Tell me: "Talk to Figma, channel {channel-id-from-logs}"

Example:
  "Talk to Figma, channel {channel-id}"

💡 Once connected, you can:
-------------------------------
- "Create a red rectangle 200x100"
- "What's currently selected in Figma?"
- "Change the selected element to blue"
- "Add rounded corners with 10px radius"

🔧 Server Management
--------------------
Check server output: Use BashOutput tool with the background shell ID
Stop server:        Use KillShell tool or: lsof -ti :3055 | xargs kill -9
Check if running:   lsof -i :3055
```

### Notes for Claude

- The server runs in the background using the Bash tool's `run_in_background: true` parameter
- You can monitor server output using the BashOutput tool with the shell ID returned when starting the server
- If the user needs to stop the server, use the KillShell tool or the kill command
- The channel ID changes each time the Figma plugin is restarted
- If port 3055 is already in use, inform the user and offer to kill the existing process

### Key Information

- MCP Installation: `~/Developer/claude-talk-to-figma-mcp`
- Plugin Manifest: `~/Developer/claude-talk-to-figma-mcp/src/claude_mcp_plugin/manifest.json`
- Server Command: `cd ~/Developer/claude-talk-to-figma-mcp && ~/.bun/bin/bun socket`
- Port: 3055
- Background Process: Use Bash tool with `run_in_background: true`

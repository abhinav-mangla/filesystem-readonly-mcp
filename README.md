# Readonly Filesystem MCP Server

Node.js server implementing Model Context Protocol (MCP) for readonly filesystem operations.

## Features

- Read files (text, media, multiple files)
- List directories and directory tree
- List directories with sizes and a summary
- Search files
- Get file metadata
- Dynamic directory access control via [Roots](https://modelcontextprotocol.io/docs/learn/client-concepts#roots)

## Directory Access Control

The server uses a flexible directory access control system. Directories can be specified via command-line arguments or dynamically via [Roots](https://modelcontextprotocol.io/docs/learn/client-concepts#roots).

### Method 1: Command-line Arguments
Specify Allowed directories when starting the server:
```bash
filesystem-readonly-mcp /path/to/dir1 /path/to/dir2
```

### Method 2: MCP Roots (Recommended)
MCP clients that support [Roots](https://modelcontextprotocol.io/docs/learn/client-concepts#roots) can dynamically update the Allowed directories. 

Roots notified by Client to Server, completely replace any server-side Allowed directories when provided.

**Important**: If server starts without command-line arguments AND client doesn't support roots protocol (or provides empty roots), the server will throw an error during initialization.

This is the recommended method, as this enables runtime directory updates via `roots/list_changed` notifications without server restart, providing a more flexible and modern integration experience.

### How It Works

The server's directory access control follows this flow:

1. **Server Startup**
   - Server starts with directories from command-line arguments (if provided)
   - If no arguments provided, server starts with empty allowed directories

2. **Client Connection & Initialization**
   - Client connects and sends `initialize` request with capabilities
   - Server checks if client supports roots protocol (`capabilities.roots`)
   
3. **Roots Protocol Handling** (if client supports roots)
   - **On initialization**: Server requests roots from client via `roots/list`
   - Client responds with its configured roots
   - Server replaces ALL allowed directories with client's roots
   - **On runtime updates**: Client can send `notifications/roots/list_changed`
   - Server requests updated roots and replaces allowed directories again

4. **Fallback Behavior** (if client doesn't support roots)
   - Server continues using command-line directories only
   - No dynamic updates possible

5. **Access Control**
   - All filesystem operations are restricted to allowed directories
   - Use `list_allowed_directories` tool to see current directories
   - Server requires at least ONE allowed directory to operate

**Note**: The server will only allow operations within directories specified either via `args` or via Roots.



## API (Read-only)

### Tools

- **read_text_file**
  - Read complete contents of a file as text
  - Inputs:
    - `path` (string)
    - `head` (number, optional): First N lines
    - `tail` (number, optional): Last N lines
  - Always treats the file as UTF-8 text regardless of extension
  - Note: `head` and `tail` cannot be used together

- **read_media_file**
  - Read an image or audio file
  - Inputs:
    - `path` (string)
  - Streams the file and returns base64 data with the corresponding MIME type

- **read_multiple_files**
  - Read multiple files simultaneously
  - Input: `paths` (string[])
  - Failed reads won't stop the entire operation

- **list_directory**
  - List directory contents with [FILE] or [DIR] prefixes
  - Input: `path` (string)

- **list_directory_with_sizes**
  - List directory contents including sizes and a summary
  - Inputs:
    - `path` (string)
    - `sortBy` ("name" | "size", optional, default: "name")
  - Output includes entries formatted with [FILE]/[DIR] and file sizes

- **directory_tree**
  - Get a recursive tree view of files and directories as JSON
  - Input: `path` (string)
  - Output: Array of entries with `name`, `type` (file|directory), and `children` for directories

- **search_files**
  - Recursively search for files/directories
  - Inputs:
    - `path` (string): Starting directory
    - `pattern` (string): Search pattern
    - `excludePatterns` (string[]): Exclude any patterns. Glob formats are supported.
  - Case-insensitive matching
  - Returns full paths to matches

- **get_file_info**
  - Get detailed file/directory metadata
  - Input: `path` (string)
  - Returns:
    - Size
    - Creation time
    - Modified time
    - Access time
    - Type (file/directory)
    - Permissions

- **list_allowed_directories**
  - List all directories the server is allowed to access
  - No input required
  - Returns:
    - Directories that this server can read from

## Usage with Claude Desktop
Add this to your `claude_desktop_config.json`:

### NPX

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "filesystem-readonly-mcp",
        "/Users/username/Desktop",
        "/path/to/other/allowed/dir"
      ]
    }
  }
}
```

## Usage with VS Code

**Method 1: User Configuration (Recommended)**
Add the configuration to your user-level MCP configuration file. Open the Command Palette (`Ctrl + Shift + P`) and run `MCP: Open User Configuration`. This will open your user `mcp.json` file where you can add the server configuration.

**Method 2: Workspace Configuration**
Alternatively, you can add the configuration to a file called `.vscode/mcp.json` in your workspace. This will allow you to share the configuration with others.

> For more details about MCP configuration in VS Code, see the [official VS Code MCP documentation](https://code.visualstudio.com/docs/copilot/mcp).


### NPX

```json
{
  "servers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "filesystem-readonly-mcp",
        "${workspaceFolder}"
      ]
    }
  }
}
```

## License

This MCP server is licensed under the MIT License. This means you are free to use, modify, and distribute the software, subject to the terms and conditions of the MIT License. For more details, please see the LICENSE file in the project repository.

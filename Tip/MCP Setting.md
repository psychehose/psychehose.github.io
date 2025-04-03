
1.  Filesystem MCP Server 
	* Node.js server implementing Model Context Protocol (MCP) for filesystem operations
	* https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem
	  
2.  Memory
	* A basic implementation of persistent memory using a local knowledge graph. This lets Claude remember information about the user across chats.
	* https://github.com/modelcontextprotocol/servers/tree/main/src/memory
	  
3.  obsidian-mcp
	* An MCP server that enables AI assistants to interact with Obsidian vaults, providing tools for reading, creating, editing and managing notes and tags.
	* https://github.com/modelcontextprotocol/servers?tab=readme-ov-file
	  
4. sequential-thinking
   * An MCP server implementation that provides a tool for dynamic and reflective problem-solving through a structured thinking process.
   * https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking
     
5. brave-search
	* An MCP server implementation that integrates the Brave Search API, providing both web and local search capabilities.
	* https://github.com/modelcontextprotocol/servers/tree/main/src/brave-search
	  


#### JSON

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "{path1}",
        "{path2}"
      ]
    },
    "memory": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-memory"
      ]
    },
    "obsidian-mcp": {
      "command": "npx",
      "args": [
        "-y",
        "obsidian-mcp",
        "{path}"
      ]
    },
    "sequential-thinking": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-sequential-thinking"
      ]
    },
    "brave-search": {
      "command": "npx",
      "args": [
        "-y",
        "@smithery/cli@latest",
        "run",
        "@smithery-ai/brave-search",
        "--key",
        "{key}"
      ]
    }
  }
}
```
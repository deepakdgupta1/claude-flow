# MCP Server

## Overview

The Model Context Protocol (MCP) server is the **primary integration point between Claude Code and Claude Flow**. It exposes Claude Flow's agent orchestration, memory, configuration, and workflow capabilities as callable tools within Claude Code sessions. This enables users to orchestrate multi-agent swarms, search vector memory, and manage configurations directly from their coding environment.

**Key Locations**:

| Path | Description |
|------|-------------|
| `v3/mcp/server.ts` | Full MCP server implementation |
| `v3/mcp/tool-registry.ts` | Tool registration and lookup |
| `v3/mcp/session-manager.ts` | Session lifecycle and timeout management |
| `v3/mcp/connection-pool.ts` | Connection pooling for efficiency |
| `v3/mcp/transport/` | Transport implementations (stdio, HTTP, WebSocket, in-process) |
| `v3/mcp/tools/` | Tool handler definitions |
| `v3/mcp/types.ts` | MCP protocol type definitions |
| `v3/src/infrastructure/mcp/` | MCP infrastructure in core module |
| `v3/@claude-flow/cli/src/mcp-server.ts` | CLI-integrated MCP server entry point |

---

## Architecture

```
Claude Code ←→ [stdio transport] ←→ MCP Server ←→ Tool Registry
                                         ↓
                                  Session Manager
                                         ↓
                              Connection Pool → Tool Handlers
                                                    ↓
                                        SwarmCoordinator / MemoryBackend / Config
```

### Request Flow

1. Claude Code sends a JSON-RPC request over stdio
2. MCP Server receives and validates the request
3. Session Manager tracks the session state
4. Tool Registry routes to the appropriate tool handler
5. Tool handler executes the operation (agent spawn, memory search, etc.)
6. Response is serialized and sent back over stdio

---

## Server Configuration

```typescript
const DEFAULT_CONFIG = {
  name: 'Claude-Flow MCP Server V3',
  version: '3.0.0',
  transport: 'stdio',        // Primary transport for Claude Code
  host: 'localhost',
  port: 3000,                // For HTTP/WebSocket transports
  enableMetrics: true,
  enableCaching: true,
  cacheTTL: 10000,           // 10 second cache TTL
  logLevel: 'info',
  requestTimeout: 30000,     // 30 second request timeout
  maxRequestSize: 10485760,  // 10 MB max request
};
```

---

## Performance Targets

| Metric | Target |
|--------|--------|
| Server startup | <400ms |
| Tool registration | <10ms |
| Tool execution overhead | <50ms |
| Request routing | <50ms |

---

## Core Components

### ToolRegistry

Fast tool registration and lookup. Tools are registered by name and expose their parameter schemas for discovery.

```typescript
interface MCPTool {
  name: string;
  description: string;
  parameters: Record<string, unknown>; // JSON Schema
}
```

Methods: `registerTool(tool)`, `registerTools(tools[])` — returns count of registered and names of failed.

### SessionManager

Manages client sessions with lifecycle tracking:
- Session creation on `initialize` request
- Session timeout handling
- Session termination (explicit or on disconnect)
- Multi-session support for concurrent clients

### ConnectionPool

Efficient resource management for tool execution:
- Pools connections to backend services
- Reduces overhead for repeated tool calls
- Configurable pool sizes

### TransportManager

Supports multiple transport protocols:

| Transport | Use Case | Protocol |
|-----------|----------|----------|
| **stdio** | Claude Code integration (default) | stdin/stdout JSON-RPC |
| **HTTP** | Remote access, REST clients | HTTP POST with JSON-RPC |
| **WebSocket** | Real-time bidirectional | WS frames with JSON-RPC |
| **in-process** | Testing, embedded use | Direct function calls |

---

## MCP Protocol Implementation

### Initialization Handshake

```
Client → Server: initialize { protocolVersion, capabilities, clientInfo }
Server → Client: { protocolVersion, capabilities, serverInfo }
Client → Server: initialized (notification)
```

### Tool Discovery

```
Client → Server: tools/list {}
Server → Client: { tools: [{ name, description, inputSchema }] }
```

### Tool Execution

```
Client → Server: tools/call { name, arguments }
Server → Client: { content: [{ type: "text", text: "..." }] }
```

### Error Handling

Standard JSON-RPC error codes plus MCP-specific codes via the `ErrorCodes` enum.

---

## Exposed Tool Categories

### AgentTools (`v3/src/infrastructure/mcp/tools/AgentTools.ts`)

| Tool | Description |
|------|-------------|
| `agent_spawn` | Create a new agent with type, capabilities, role |
| `agent_list` | List all active agents |
| `agent_terminate` | Terminate an agent by ID |
| `agent_status` | Get agent status and metadata |
| `agent_metrics` | Get agent performance metrics |

### MemoryTools (`v3/src/infrastructure/mcp/tools/MemoryTools.ts`)

| Tool | Description |
|------|-------------|
| `memory_store` | Store content with optional embedding |
| `memory_retrieve` | Retrieve by ID |
| `memory_search` | Semantic vector search with similarity scores |
| `memory_query` | Structured query by agentId, type, time range |

### ConfigTools (`v3/src/infrastructure/mcp/tools/ConfigTools.ts`)

| Tool | Description |
|------|-------------|
| `config_get` | Get current configuration |
| `config_set` | Update configuration values |
| `config_validate` | Validate configuration |

### Additional Tools (via Hooks MCP integration)

The hooks system exposes additional tools: `pre_edit`, `post_edit`, `route_task`, `metrics`, `pre_command`, `post_command`, `daemon_status`, `statusline`, and all worker management tools.

---

## Integration with Claude Code

### Setup

```bash
# Register Claude Flow as an MCP server in Claude Code
claude mcp add claude-flow -- npx -y @claude-flow/cli@alpha
```

### Auto-Detection

When the CLI detects that stdin is piped (as it is when invoked by Claude Code as an MCP server), it automatically starts in MCP server mode. This eliminates the need for a separate server process.

### Usage in Claude Code

Once registered, all Claude Flow tools appear as callable functions. Claude Code can:
- Spawn and coordinate agent swarms
- Store and retrieve patterns from vector memory
- Execute hooks for learning and optimization
- Manage sessions across conversations

---

## Server Interface

```typescript
interface IMCPServer {
  start(): Promise<void>;
  stop(): Promise<void>;
  registerTool(tool: MCPTool): boolean;
  registerTools(tools: MCPTool[]): { registered: number; failed: string[] };
  getHealthStatus(): Promise<{ healthy: boolean; error?: string; metrics?: Record<string, number> }>;
  getMetrics(): MCPServerMetrics;
  getSessions(): MCPSession[];
  getSession(sessionId: string): MCPSession | undefined;
  terminateSession(sessionId: string): boolean;
}
```

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| stdio as default transport | Native to Claude Code's MCP integration model |
| Auto-detection of MCP mode | Single binary serves both CLI and MCP modes |
| Connection pooling | Reduces overhead for high-frequency tool calls |
| <50ms execution overhead target | Tools must feel instant in interactive Claude Code sessions |
| Tool registry pattern | Plugins can dynamically register additional tools |
| Session management | Enables stateful interactions across multiple tool calls |
| Caching (10s TTL) | Reduces redundant backend calls for repeated queries |
| 10MB max request size | Prevents memory exhaustion from oversized payloads |

# Module 13: Claims-based Authorization

**Location**: `v3/@claude-flow/claims/`

**Purpose**: Claims-based authorization system for managing permissions between human users and AI agents. Determines what actions agents are allowed to perform, enforcing security boundaries in multi-agent systems.

---

## Overview

The claims system provides a flexible authorization layer that controls agent behavior at runtime. Rather than hardcoding permissions, claims are granted and revoked dynamically, enabling fine-grained control over what agents can do.

A **claim** is a key-value assertion about what an entity (agent or user) is permitted to do. Claims are checked before sensitive operations and can be scoped by resource, action, or context.

```
┌──────────────┐     check      ┌──────────────────┐
│    Agent      │──────────────▶│  Claims Engine    │
│  (requests    │               │                   │
│   action)     │◀──────────────│  grant / deny     │
└──────────────┘                └──────────────────┘
                                       ▲
                                       │ manage
                                ┌──────┴───────┐
                                │  Human User  │
                                │  or Admin    │
                                └──────────────┘
```

---

## CLI Commands

### `claims check`

Check if an agent has a specific claim or permission.

```bash
npx claude-flow@v3alpha claims check --agent <agent-id> --claim <claim-name>
```

**Arguments**:
- `--agent`: The agent ID to check permissions for
- `--claim`: The claim name to verify (e.g., `file:write`, `service:deploy`)

**Output**: Returns `granted` or `denied` with the matching rule details.

**Example**:
```bash
npx claude-flow@v3alpha claims check --agent coder-01 --claim "file:write:/src/**"
# Output: GRANTED - Matched rule: "coder agents can write to /src/"

npx claude-flow@v3alpha claims check --agent tester-01 --claim "file:write:/src/**"
# Output: DENIED - No matching grant found
```

### `claims grant`

Grant a claim to an agent.

```bash
npx claude-flow@v3alpha claims grant --agent <agent-id> --claim <claim-name> [--scope <scope>] [--expires <duration>]
```

**Arguments**:
- `--agent`: The agent ID to grant the claim to
- `--claim`: The claim to grant
- `--scope`: Optional scope restriction (e.g., a file path pattern, service name)
- `--expires`: Optional expiration duration (e.g., `1h`, `30m`, `7d`)

**Examples**:
```bash
# Grant file write access to a specific directory
npx claude-flow@v3alpha claims grant --agent coder-01 --claim "file:write" --scope "/src/**"

# Grant temporary deploy access
npx claude-flow@v3alpha claims grant --agent deployer-01 --claim "service:deploy" --expires 1h

# Grant broad read access
npx claude-flow@v3alpha claims grant --agent researcher-01 --claim "file:read" --scope "**/*"
```

### `claims revoke`

Revoke a claim from an agent.

```bash
npx claude-flow@v3alpha claims revoke --agent <agent-id> --claim <claim-name> [--scope <scope>]
```

**Arguments**:
- `--agent`: The agent ID to revoke the claim from
- `--claim`: The claim to revoke
- `--scope`: Optional scope to narrow the revocation

**Examples**:
```bash
# Revoke all file write access
npx claude-flow@v3alpha claims revoke --agent coder-01 --claim "file:write"

# Revoke access to a specific scope only
npx claude-flow@v3alpha claims revoke --agent coder-01 --claim "file:write" --scope "/config/**"
```

### `claims list`

List all claims for an agent or user.

```bash
npx claude-flow@v3alpha claims list [--agent <agent-id>] [--claim <claim-pattern>] [--format <json|table>]
```

**Arguments**:
- `--agent`: Filter by agent ID (optional — lists all if omitted)
- `--claim`: Filter by claim name pattern (supports wildcards)
- `--format`: Output format, defaults to `table`

**Examples**:
```bash
# List all claims for a specific agent
npx claude-flow@v3alpha claims list --agent coder-01

# List all file-related claims across all agents
npx claude-flow@v3alpha claims list --claim "file:*"

# JSON output for programmatic use
npx claude-flow@v3alpha claims list --format json
```

---

## Claim Structure

Claims follow a hierarchical naming convention:

```
<domain>:<action>[:<resource>]
```

### Common Claim Domains

| Domain | Description | Example Claims |
|---|---|---|
| `file` | File system operations | `file:read`, `file:write`, `file:delete` |
| `service` | External service access | `service:deploy`, `service:restart`, `service:config` |
| `agent` | Agent management | `agent:spawn`, `agent:terminate`, `agent:configure` |
| `memory` | Memory operations | `memory:read`, `memory:write`, `memory:delete` |
| `task` | Task management | `task:create`, `task:assign`, `task:cancel` |
| `network` | Network access | `network:http`, `network:webhook`, `network:dns` |
| `exec` | Command execution | `exec:shell`, `exec:script`, `exec:build` |

### Scope Patterns

Scopes use glob-style patterns for resource matching:

```
/src/**          → All files under /src/
/config/*.json   → JSON files in /config/
*.test.ts        → All test files
production       → Production environment
api-service      → Specific service name
```

---

## Use Cases

### Restricting File Write Access

Ensure only coder agents can write to source files, while reviewers have read-only access:

```bash
# Coders can write source files
npx claude-flow@v3alpha claims grant --agent coder-01 --claim "file:write" --scope "/src/**"

# Reviewers can only read
npx claude-flow@v3alpha claims grant --agent reviewer-01 --claim "file:read" --scope "**/*"

# No one writes to config without explicit grant
# (default deny — no grant means no access)
```

### Controlling External Service Access

Limit which agents can interact with external services:

```bash
# Only deployer agents can trigger deployments
npx claude-flow@v3alpha claims grant --agent deployer-01 --claim "service:deploy" --scope "staging"

# Temporary production access with expiration
npx claude-flow@v3alpha claims grant --agent deployer-01 --claim "service:deploy" --scope "production" --expires 30m
```

### Managing Human-Agent Coordination

Human users can dynamically adjust agent permissions during a session:

```bash
# Start with restricted access
npx claude-flow@v3alpha claims grant --agent assistant-01 --claim "file:read"

# Expand permissions after review
npx claude-flow@v3alpha claims grant --agent assistant-01 --claim "file:write" --scope "/src/feature/**"

# Revoke when done
npx claude-flow@v3alpha claims revoke --agent assistant-01 --claim "file:write"
```

### Enforcing Security Boundaries

Prevent agents from escalating their own privileges:

```bash
# Only coordinators can spawn new agents
npx claude-flow@v3alpha claims grant --agent coordinator-01 --claim "agent:spawn"

# Agents cannot modify their own claims
# (enforced at the engine level — self-grant is always denied)
```

---

## Integration with Security Module

The claims system integrates with the `@claude-flow/security` module for enforcement:

```
┌─────────────┐     action     ┌──────────────┐     check     ┌──────────────┐
│   Agent      │──────────────▶│   Security    │──────────────▶│   Claims     │
│   Runtime    │               │   Module      │               │   Engine     │
│              │◀──────────────│ (enforcement) │◀──────────────│ (decision)   │
│              │  allow/deny   │              │   grant/deny  │              │
└─────────────┘               └──────────────┘               └──────────────┘
```

Security enforcement points:
- **InputValidator**: Checks `exec:*` claims before command execution
- **PathValidator**: Checks `file:*` claims before file system operations
- **SafeExecutor**: Checks `exec:shell` claims before shell command execution

---

## Programmatic API

Claims can also be managed programmatically within agent code:

```typescript
import { ClaimsEngine } from '@claude-flow/claims';

const claims = new ClaimsEngine();

// Check permission
const allowed = await claims.check('coder-01', 'file:write', '/src/index.ts');
if (!allowed) {
  throw new Error('Agent lacks file:write permission');
}

// Grant with options
await claims.grant('coder-01', 'file:write', {
  scope: '/src/**',
  expires: '1h',
  grantedBy: 'coordinator-01',
});

// Revoke
await claims.revoke('coder-01', 'file:write', { scope: '/config/**' });

// List
const agentClaims = await claims.list({ agent: 'coder-01' });
```

---

## Design Decisions

### Claims-based Model
The claims-based model was chosen over traditional RBAC (Role-Based Access Control) for its flexibility. Claims can express both role-based patterns ("all coders can write code") and attribute-based patterns ("this specific agent can write to this specific path for the next 30 minutes"). This dual capability avoids the rigidity of pure RBAC without the complexity of full ABAC (Attribute-Based Access Control).

### Human-Agent Coordination
The system explicitly supports human users managing agent permissions, not just agent-to-agent authorization. This reflects the reality of AI agent systems where a human operator needs to dynamically expand or restrict agent capabilities based on context and trust.

### Default Deny
The system follows a default-deny policy — if no matching grant exists for a requested action, the action is denied. This is the safer default for AI agent systems where unexpected behavior should be constrained rather than permitted.

### Self-Grant Prevention
Agents cannot grant claims to themselves. This prevents privilege escalation attacks where a compromised or misbehaving agent attempts to expand its own permissions. Only coordinators or human users can modify claims.

### Integration with Security Module
Claims are enforced through the security module rather than being checked ad-hoc throughout the codebase. This centralized enforcement ensures consistent policy application and makes security auditing straightforward.

# Security

## Overview

The Security module provides comprehensive defense-in-depth addressing identified CVEs and common attack vectors. It offers input validation, path traversal prevention, command injection blocking, password hashing, credential generation, and token management as independently usable components that can also be composed into a unified security posture.

**Location**: `v3/@claude-flow/security/`

---

## CVE Remediations

### CVE-2: Weak Password Hashing — `PasswordHasher`

Replaces any weak or custom hashing with industry-standard bcrypt.

```typescript
import { PasswordHasher } from '@claude-flow/security';

const hasher = new PasswordHasher({ rounds: 12 });

const hash = await hasher.hash(password);
const isValid = await hasher.verify(password, hash);
```

| Constant | Value | Description |
|----------|-------|-------------|
| `MIN_BCRYPT_ROUNDS` | 12 | Minimum acceptable bcrypt cost factor |
| `MAX_PASSWORD_LENGTH` | 72 | bcrypt's maximum input length (bytes) |

**Configuration**: Rounds are configurable (recommended: 12–14). Higher values increase computation time exponentially, providing stronger resistance to brute-force attacks.

---

### CVE-3: Hardcoded Default Credentials — `CredentialGenerator`

Eliminates hardcoded defaults by generating cryptographically secure credentials at runtime.

```typescript
import { CredentialGenerator } from '@claude-flow/security';

const generator = new CredentialGenerator();
const credentials = generator.generateCredentials();
// { apiKey: "cf_...", password: "..." }
```

- All generated values use `crypto.randomBytes()` for entropy
- No default passwords, API keys, or tokens exist anywhere in the codebase
- Generated credentials meet minimum complexity requirements

---

### HIGH-1: Command Injection — `SafeExecutor`

Prevents command injection via a whitelist-based execution model. Only explicitly allowed commands can be executed.

```typescript
import { SafeExecutor, createDevelopmentExecutor, createReadOnlyExecutor } from '@claude-flow/security';

// Custom executor with specific allowed commands
const executor = new SafeExecutor({
  allowedCommands: ['git', 'npm', 'npx', 'node']
});

const result = await executor.execute('git', ['status']);
```

**Default allowed commands**: `git`, `npm`, `npx`, `node`

**Argument sanitization**: All arguments are validated and sanitized before execution. Shell metacharacters, command chaining operators (`&&`, `||`, `;`, `` ` ``), and other injection vectors are blocked.

**Streaming support**: The executor supports streaming stdout/stderr for long-running commands.

**Presets**:

| Preset | Commands | Use Case |
|--------|----------|----------|
| `createDevelopmentExecutor()` | git, npm, npx, node, tsc, eslint | Local development |
| `createReadOnlyExecutor()` | git (read-only subcommands only) | CI/CD, auditing |

---

### HIGH-2: Path Traversal — `PathValidator`

Validates file paths against a set of allowed prefixes, blocking directory traversal attacks.

```typescript
import { PathValidator, createProjectPathValidator } from '@claude-flow/security';

const validator = createProjectPathValidator('/workspace/my-project');

const result = validator.validate('../../../etc/passwd');
// { valid: false, reason: 'Path traversal detected' }

const result2 = validator.validate('src/index.ts');
// { valid: true, resolvedPath: '/workspace/my-project/src/index.ts' }
```

**Blocked patterns**:
- `..` directory traversal sequences
- Null bytes (`\0`)
- Paths resolving outside allowed prefixes

**Return type**: `PathValidationResult` containing `valid: boolean`, `resolvedPath?: string`, and `reason?: string`.

**Presets**:

| Preset | Allowed Paths | Use Case |
|--------|---------------|----------|
| `createProjectPathValidator(root)` | `root/src`, `root/config`, `root/tests` | Source code access only |
| `createFullProjectPathValidator(root)` | Entire `root` directory | Full project access |

---

## Input Validation — `InputValidator`

Built on Zod schemas, the InputValidator provides runtime type validation at system boundaries.

### Base Schemas

| Schema | Description |
|--------|-------------|
| `SafeString` | Sanitized string (no control characters, length-limited) |
| `Identifier` | Alphanumeric + underscore/hyphen identifiers |
| `Filename` | Valid filename (no path separators, no traversal) |
| `Email` | RFC-compliant email address |
| `Password` | Minimum complexity password |
| `UUID` | UUID v4 format |
| `HttpsUrl` | HTTPS-only URL |
| `Semver` | Semantic versioning string |
| `Port` | Valid port number (1–65535) |
| `IPv4` | IPv4 address |

### Domain Schemas

**Authentication**:
- `UserRole` — Enumerated user roles
- `Permission` — Permission identifiers
- `LoginRequest` — Login payload (email + password)
- `CreateUser` — User creation payload
- `CreateApiKey` — API key creation payload

**Agent/Task**:
- `AgentType` — Valid agent type identifiers
- `SpawnAgent` — Agent spawn request
- `TaskInput` — Task creation input

**Command/Path**:
- `CommandArgument` — Validated command-line argument
- `Path` — Validated file path

**Configuration**:
- `SecurityConfig` — Security module configuration
- `ExecutorConfig` — SafeExecutor configuration

### Sanitization Functions

```typescript
import { sanitizeString, sanitizeHtml, sanitizePath } from '@claude-flow/security';

const clean = sanitizeString(userInput);     // Remove control chars, trim, limit length
const safeHtml = sanitizeHtml(htmlContent);  // Strip dangerous tags and attributes
const safePath = sanitizePath(filePath);     // Normalize and validate path
```

### Constants

- `PATTERNS` — Compiled regex patterns for common validation (email, UUID, semver, etc.)
- `LIMITS` — Maximum length constants for strings, filenames, paths, etc.

---

## Token Generation — `TokenGenerator`

Secure token generation for authentication, sessions, and verification.

```typescript
import { TokenGenerator, getDefaultGenerator, quickGenerate } from '@claude-flow/security';

// Full instance with HMAC signing
const generator = new TokenGenerator({ secret: process.env.HMAC_SECRET });

const token = generator.generateToken();           // Random secure token
const signed = generator.generateSignedToken(data); // HMAC-signed token
const code = generator.generateVerificationCode();  // Numeric verification code
const isValid = generator.verifySignedToken(signed); // Verify HMAC signature

// Convenience functions
const oneOff = quickGenerate();                    // One-off token generation
const singleton = getDefaultGenerator();           // Singleton access
```

| Constant | Value | Description |
|----------|-------|-------------|
| `DEFAULT_TOKEN_EXPIRATION` | 3600s (1 hour) | Default token TTL |
| `DEFAULT_SESSION_EXPIRATION` | 86400s (24 hours) | Default session TTL |

---

## Security Module Factory

The `createSecurityModule()` function composes all security components into a single configured module.

```typescript
import { createSecurityModule, auditSecurityConfig } from '@claude-flow/security';

// Audit configuration before use
const warnings = auditSecurityConfig(config);
if (warnings.length > 0) {
  console.warn('Security config issues:', warnings);
}

// Create the complete module
const security = createSecurityModule({
  projectRoot: '/workspace/my-project',
  hmacSecret: process.env.HMAC_SECRET,
  bcryptRounds: 12,
  allowedCommands: ['git', 'npm', 'npx', 'node']
});

// All components are now available
security.passwordHasher.hash(password);
security.credentialGenerator.generateCredentials();
security.executor.execute('git', ['status']);
security.pathValidator.validate('src/index.ts');
security.inputValidator.validate(LoginRequest, payload);
security.tokenGenerator.generateToken();
```

### Configuration

| Field | Type | Description |
|-------|------|-------------|
| `projectRoot` | `string` | Root directory for path validation |
| `hmacSecret` | `string` | Secret for HMAC token signing |
| `bcryptRounds` | `number` | bcrypt cost factor (default: 12) |
| `allowedCommands` | `string[]` | Whitelist for SafeExecutor |

### Security Audit

`auditSecurityConfig(config)` checks for common misconfigurations and returns an array of warning strings:

- bcrypt rounds below minimum
- Missing HMAC secret
- Overly permissive command whitelist
- Project root pointing to system directories

---

## AIDefence Module

**Location**: `v3/@claude-flow/aidefence/`

The AIDefence module addresses AI-specific security concerns:

| Feature | Description |
|---------|-------------|
| Prompt injection detection | Identifies and blocks prompt injection attempts in agent inputs |
| Security scanning | Scans agent prompts and outputs for known attack patterns |
| Agent boundary enforcement | Ensures agents operate within their defined security perimeter |

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| Defense-in-depth | Multiple independent security layers ensure no single bypass compromises the system |
| Zod schemas for validation | Runtime type validation at system boundaries catches malformed input that TypeScript's compile-time checks cannot |
| Whitelist for commands | Deny-by-default is the only safe approach for command execution; blacklists inevitably miss edge cases |
| bcrypt for passwords | Industry standard, timing-safe comparison, configurable work factor that scales with hardware improvements |
| Separate factory presets | Development and read-only presets provide appropriate security postures without manual configuration |
| Independent components | Each security component (hasher, validator, executor, etc.) is independently usable for targeted hardening, or composed via `createSecurityModule` for comprehensive coverage |
| AI-specific defenses | LLM-powered agents face unique attack vectors (prompt injection, jailbreaking) that traditional security tools do not address |

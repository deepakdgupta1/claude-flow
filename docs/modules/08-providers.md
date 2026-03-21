# Module 08: Providers

**Location**: `v3/@claude-flow/providers/`

**Purpose**: Multi-LLM provider system with load balancing, failover, cost optimization, and health monitoring.

---

## Overview

The Providers module abstracts interactions with multiple LLM providers behind a unified interface. It enables intelligent routing of requests based on task complexity, cost constraints, and provider health. The system supports both cloud-hosted models and local deployments, making it suitable for both standard and air-gapped environments.

---

## Supported Providers

### AnthropicProvider

Integration with Anthropic's Claude model family.

| Model | Use Case |
|-------|----------|
| Claude 3.5 Sonnet | Balanced performance and cost for most tasks |
| Claude 3 Opus | Complex reasoning, architecture, security analysis |
| Claude 3 Sonnet | Mid-tier tasks, code generation |
| Claude 3 Haiku | Simple tasks, formatting, quick lookups |

### OpenAIProvider

Integration with OpenAI's GPT model family.

| Model | Use Case |
|-------|----------|
| GPT-4o | Multi-modal tasks, complex reasoning |
| o1 | Deep reasoning, mathematical and logical problems |
| GPT-4 | General-purpose complex tasks |
| GPT-3.5-turbo | Simple tasks, high-throughput scenarios |

### GoogleProvider

Integration with Google's Gemini model family.

| Model | Use Case |
|-------|----------|
| Gemini 2.0 | Latest-generation multi-modal tasks |
| Gemini 1.5 Pro | Long-context tasks, document analysis |
| Gemini 1.5 Flash | Fast, cost-effective general tasks |

### CohereProvider

Integration with Cohere's Command model family.

| Model | Use Case |
|-------|----------|
| Command R+ | Complex reasoning and generation |
| Command R | General-purpose generation |
| Command Light | Simple tasks, classification |

### OllamaProvider

Integration with locally-hosted models via Ollama. Enables air-gapped deployments with no external API dependencies.

| Model | Use Case |
|-------|----------|
| Llama | General-purpose local inference |
| Mistral | Efficient local code and text generation |
| CodeLlama | Code-specific local tasks |
| Phi | Lightweight local inference |

### RuVectorProvider

Integration with the RuVector intelligence system for pattern-based reasoning and neural learning capabilities.

---

## BaseProvider

Abstract base class that all providers must extend. Ensures a consistent interface across all provider implementations.

```typescript
abstract class BaseProvider {
  constructor(options: BaseProviderOptions);

  abstract complete(prompt: string, options?: CompletionOptions): Promise<CompletionResult>;
  abstract chat(messages: Message[], options?: ChatOptions): Promise<ChatResult>;
  abstract isAvailable(): Promise<boolean>;
  abstract getModels(): ModelInfo[];

  protected logger: ILogger;
  protected config: BaseProviderOptions;
}
```

### BaseProviderOptions

| Option | Type | Description |
|--------|------|-------------|
| `apiKey` | `string` | API key for the provider |
| `baseUrl` | `string?` | Custom API endpoint (for proxies or local deployments) |
| `timeout` | `number?` | Request timeout in milliseconds |
| `maxRetries` | `number?` | Maximum retry attempts on failure |
| `logger` | `ILogger?` | Logger instance (defaults to `consoleLogger`) |

### ILogger Interface

Logging abstraction used across all providers.

```typescript
interface ILogger {
  debug(message: string, ...args: unknown[]): void;
  info(message: string, ...args: unknown[]): void;
  warn(message: string, ...args: unknown[]): void;
  error(message: string, ...args: unknown[]): void;
}
```

The default implementation (`consoleLogger`) writes to `console.*` methods.

---

## ProviderManager

Central manager for all registered providers. Handles routing, load balancing, failover, and health monitoring.

### Factory Function

```typescript
const manager = createProviderManager({
  providers: [
    { type: 'anthropic', apiKey: process.env.ANTHROPIC_API_KEY },
    { type: 'openai', apiKey: process.env.OPENAI_API_KEY },
    { type: 'ollama', baseUrl: 'http://localhost:11434' },
  ],
  strategy: 'cost-based',
  enableCircuitBreaker: true,
  enableCaching: true,
});
```

### Load Balancing Strategies

| Strategy | Description |
|----------|-------------|
| `round-robin` | Distributes requests evenly across providers in order |
| `latency-based` | Routes to the provider with the lowest recent response time |
| `cost-based` | Routes to the cheapest provider capable of handling the task complexity |

### Automatic Failover

When a provider fails to respond or returns an error, the ProviderManager automatically routes the request to the next available provider in the chain.

**Failover sequence**:
1. Primary provider receives request
2. On failure, request is retried based on `maxRetries` configuration
3. If retries are exhausted, request is routed to the next available provider
4. Process continues until a provider succeeds or all providers are exhausted
5. If all providers fail, a `ProviderExhaustionError` is thrown

### Request Caching

Caches responses for identical requests to reduce redundant API calls. Cache keys are computed from the request content hash.

- **Cache TTL**: Configurable per-provider (default: 5 minutes)
- **Cache invalidation**: Automatic on TTL expiry; manual via `manager.clearCache()`
- **Cache scope**: Per-provider to handle model-specific response variations

### Cost Optimization

Achieves 85%+ cost savings through intelligent request routing.

**Routing logic**:
1. Estimate task complexity from prompt length, context requirements, and task type
2. Select the cheapest model tier capable of handling the estimated complexity
3. Route to the selected model via the appropriate provider

| Complexity | Routed To | Approximate Cost |
|------------|-----------|-----------------|
| Low (formatting, simple fixes) | Haiku / GPT-3.5-turbo / Flash | ~$0.0002/request |
| Medium (features, refactoring) | Sonnet / GPT-4o / Pro | ~$0.003/request |
| High (architecture, security) | Opus / o1 / Gemini 2.0 | ~$0.015/request |

### Circuit Breaker

Prevents cascade failures when a provider is experiencing issues.

**States**:
- **Closed** (normal) — Requests flow through normally. Failures are counted.
- **Open** (tripped) — All requests are immediately routed to fallback providers. No requests sent to the failing provider.
- **Half-Open** (testing) — A single test request is sent to the provider. If it succeeds, the circuit closes. If it fails, the circuit reopens.

**Configuration**:
| Parameter | Default | Description |
|-----------|---------|-------------|
| `failureThreshold` | 5 | Failures before circuit opens |
| `resetTimeout` | 30s | Time before transitioning to half-open |
| `successThreshold` | 2 | Successes in half-open before closing |

### Health Monitoring

Continuously tracks provider availability and performance metrics.

**Tracked Metrics**:
- Response time (average, p50, p95, p99)
- Error rate (rolling window)
- Availability (uptime percentage)
- Request throughput
- Cost per request

Health data feeds into the load balancing strategy and circuit breaker decisions.

---

## Design Decisions

1. **Abstract base class** — `BaseProvider` ensures a consistent interface across all providers. New providers can be added by extending the base class without modifying the manager.

2. **Provider-agnostic manager** — The `ProviderManager` operates on the `BaseProvider` interface, enabling providers to be swapped, added, or removed without code changes to consuming modules.

3. **Cost-based routing** — The default routing strategy prioritizes cheaper models for simpler tasks. This is the primary mechanism for the 85%+ cost savings claim, routing the majority of simple operations to Haiku-tier models.

4. **Circuit breaker pattern** — Prevents the system from overwhelming a failing provider with retry requests, which would degrade the provider further and waste resources.

5. **Local model support (Ollama)** — The OllamaProvider enables fully air-gapped deployments where no external API calls are made. Models run locally with the same interface as cloud providers.

6. **Pluggable logging** — The `ILogger` interface allows providers to integrate with any logging framework. The default `consoleLogger` works out of the box with no configuration.

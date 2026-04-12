# lm15 Type Definitions

Canonical, language-agnostic type definitions for all lm15 implementations.

All types are **immutable value objects** (frozen dataclasses in Python, `readonly` interfaces in TypeScript, etc.). Implementations must enforce the noted constraints.

---

## Scalars & Aliases

```
JsonPrimitive  = null | bool | int | float | string
JsonValue      = JsonPrimitive | JsonValue[] | { [key: string]: JsonValue }
JsonArray      = JsonValue[]
JsonObject     = { [key: string]: JsonValue }
```

## Enums (string literals)

### Role
```
"user" | "assistant" | "tool"
```

### PartType
```
"text" | "image" | "audio" | "video" | "document"
| "tool_call" | "tool_result" | "thinking" | "refusal" | "citation"
```

### ToolType
```
"function" | "builtin"
```

### ReasoningEffort
```
"low" | "medium" | "high"
```

### FinishReason
```
"stop" | "length" | "tool_call" | "content_filter" | "error"
```

### DataSourceType
```
"base64" | "url" | "file"
```

### StreamEventType
```
"start" | "delta" | "part_start" | "part_end" | "end" | "error"
```

### PartDeltaType
```
"text" | "tool_call" | "thinking" | "audio"
```

### ErrorCode
```
"auth" | "billing" | "rate_limit" | "invalid_request"
| "context_length" | "timeout" | "server" | "provider"
```

### AudioEncoding
```
"pcm16" | "opus" | "mp3" | "aac"
```

---

## Core Types

### DataSource

| Field | Type | Required | Constraints |
|---|---|---|---|
| `type` | DataSourceType | ✅ | |
| `media_type` | string? | `base64` ✅ | |
| `data` | string? | `base64` ✅ | base64-encoded bytes |
| `url` | string? | `url` ✅ | |
| `file_id` | string? | `file` ✅ | |
| `detail` | `"low"` \| `"high"` \| `"auto"` ? | | |

**Constraint:** Validation depends on `type`:
- `base64` → `data` and `media_type` required
- `url` → `url` required
- `file` → `file_id` required

**Derived property:** `bytes` → base64-decode `data`. Only valid for `type="base64"`.

### Part (base)

Abstract base with subtype dispatch on `type`. All Part subtypes share a common accessor interface:

| Accessor | Default |
|---|---|
| `text` | null |
| `source` | null |
| `id` | null |
| `name` | null |
| `input` | null |
| `content` | () |
| `is_error` | null |
| `redacted` | null |
| `summary` | null |
| `url` | null |
| `title` | null |
| `metadata` | null |

Each subtype only defines the fields it uses. Accessing undefined fields returns the default above (not an error).

### TextPart (type = "text")

| Field | Type | Required |
|---|---|---|
| `text` | string | ✅ |
| `metadata` | JsonObject? | |

### ThinkingPart (type = "thinking")

| Field | Type | Required |
|---|---|---|
| `text` | string | ✅ |
| `redacted` | bool? | |
| `summary` | string? | |
| `metadata` | JsonObject? | |

### RefusalPart (type = "refusal")

| Field | Type | Required |
|---|---|---|
| `text` | string | ✅ |

### CitationPart (type = "citation")

| Field | Type | Required |
|---|---|---|
| `text` | string? | |
| `url` | string? | |
| `title` | string? | |

### ImagePart (type = "image")

| Field | Type | Required |
|---|---|---|
| `source` | DataSource | ✅ |
| `metadata` | JsonObject? | |

### AudioPart (type = "audio")

| Field | Type | Required |
|---|---|---|
| `source` | DataSource | ✅ |
| `metadata` | JsonObject? | |

### VideoPart (type = "video")

| Field | Type | Required |
|---|---|---|
| `source` | DataSource | ✅ |
| `metadata` | JsonObject? | |

### DocumentPart (type = "document")

| Field | Type | Required |
|---|---|---|
| `source` | DataSource | ✅ |
| `metadata` | JsonObject? | |

### ToolCallPart (type = "tool_call")

| Field | Type | Required |
|---|---|---|
| `id` | string | ✅ |
| `name` | string | ✅ |
| `input` | JsonObject | ✅ |

### ToolResultPart (type = "tool_result")

| Field | Type | Required |
|---|---|---|
| `id` | string | ✅ |
| `name` | string? | |
| `content` | Part[] | |
| `is_error` | bool? | |

### Part Factory Methods

Implementations should provide these static constructors:

- `Part.text_part(text)` → TextPart
- `Part.thinking(text, redacted?, summary?, metadata?)` → ThinkingPart
- `Part.refusal(text)` → RefusalPart
- `Part.citation(text?, url?, title?)` → CitationPart
- `Part.image(url? | data? | file_id?, media_type?, detail?, cache?)` → ImagePart
- `Part.audio(url? | data? | file_id?, media_type?, detail?, cache?)` → AudioPart
- `Part.video(url? | data? | file_id?, media_type?, detail?, cache?)` → VideoPart
- `Part.document(url? | data? | file_id?, media_type?, detail?, cache?)` → DocumentPart
- `Part.tool_call(id, name, input)` → ToolCallPart
- `Part.tool_result(id, content, is_error?, name?)` → ToolResultPart

Media part factories (`image`, `audio`, `video`, `document`) require exactly one of `url`, `data`, `file_id`. When `data` is raw bytes, base64-encode automatically.

---

## Tool Types

### FunctionTool (type = "function")

| Field | Type | Required | Default |
|---|---|---|---|
| `name` | string | ✅ | |
| `description` | string? | | |
| `parameters` | JsonObject? | | `{"type": "object", "properties": {}}` |
| `fn` | callable? | | Not serialized |

### BuiltinTool (type = "builtin")

| Field | Type | Required |
|---|---|---|
| `name` | string | ✅ |
| `description` | string? | |
| `builtin_config` | JsonObject? | |

### ToolCallInfo

| Field | Type | Required |
|---|---|---|
| `id` | string | ✅ |
| `name` | string | ✅ |
| `input` | JsonObject | ✅ |

### ToolConfig

| Field | Type | Default |
|---|---|---|
| `mode` | `"auto"` \| `"required"` \| `"none"` | `"auto"` |
| `allowed` | string[] | `[]` |
| `parallel` | bool? | |

---

## Configuration

### ReasoningConfig

| Field | Type | Required | Constraints |
|---|---|---|---|
| `enabled` | bool | ✅ | |
| `budget` | int? | | > 0 |
| `effort` | ReasoningEffort? | | |

### Config

| Field | Type | Default | Constraints |
|---|---|---|---|
| `max_tokens` | int? | | > 0 |
| `temperature` | float? | | >= 0 |
| `top_p` | float? | | [0, 1] |
| `top_k` | int? | | |
| `stop` | string[] | `[]` | |
| `response_format` | JsonObject? | | |
| `tool_config` | ToolConfig? | | |
| `reasoning` | ReasoningConfig? | | |
| `provider` | JsonObject? | | Passthrough to provider |

### AudioFormat

| Field | Type | Required | Constraints |
|---|---|---|---|
| `encoding` | AudioEncoding | ✅ | |
| `sample_rate` | int | ✅ | > 0 |
| `channels` | int | 1 | > 0 |

---

## Messages

### Message

| Field | Type | Required | Constraints |
|---|---|---|---|
| `role` | Role | ✅ | |
| `parts` | Part[] | ✅ | non-empty |
| `name` | string? | | |

Factory methods:
- `Message.user(text)` → Message(role="user", parts=[TextPart(text)])
- `Message.assistant(text)` → Message(role="assistant", parts=[TextPart(text)])
- `Message.tool_results(results: {id: string | Part | Part[]})` → Message(role="tool", parts=[...])

---

## Request / Response

### LMRequest

| Field | Type | Required | Constraints |
|---|---|---|---|
| `model` | string | ✅ | non-empty |
| `messages` | Message[] | ✅ | non-empty |
| `system` | string \| Part[]? | | if Part[], non-empty |
| `tools` | Tool[] | `[]` | |
| `config` | Config | `Config()` | |

### Usage

| Field | Type | Default |
|---|---|---|
| `input_tokens` | int | 0 |
| `output_tokens` | int | 0 |
| `total_tokens` | int | 0 |
| `cache_read_tokens` | int? | |
| `cache_write_tokens` | int? | |
| `reasoning_tokens` | int? | |
| `input_audio_tokens` | int? | |
| `output_audio_tokens` | int? | |

### LMResponse

| Field | Type | Required |
|---|---|---|
| `id` | string | ✅ |
| `model` | string | ✅ |
| `message` | Message | ✅ |
| `finish_reason` | FinishReason | ✅ |
| `usage` | Usage | ✅ |
| `provider` | JsonObject? | |

Derived properties:
- `text` → join text parts with "\n", null if none
- `thinking` → join thinking parts with "\n", null if none
- `tool_calls` → list of ToolCallPart from message.parts
- `image` → first ImagePart or null
- `images` → all ImageParts
- `audio` → first AudioPart or null
- `citations` → all CitationParts
- `json` → parse `text` as JSON, error if invalid
- `image_bytes` → decode first image's base64 data
- `audio_bytes` → decode first audio's base64 data

---

## Streaming

### ErrorInfo

| Field | Type | Required |
|---|---|---|
| `code` | ErrorCode | ✅ |
| `message` | string | ✅ |
| `provider_code` | string? | |

### PartDelta

| Field | Type | Required |
|---|---|---|
| `type` | PartDeltaType | ✅ |
| `text` | string? | required if type ∈ {"text", "thinking"} |
| `data` | string? | required if type = "audio" |
| `input` | string? | required if type = "tool_call" |

### StreamEvent

| Field | Type | Required | Constraints |
|---|---|---|---|
| `type` | StreamEventType | ✅ | |
| `id` | string? | | |
| `model` | string? | | |
| `part_index` | int? | | |
| `delta` | PartDelta \| JsonObject? | | required if type = "delta" |
| `part_type` | string? | | |
| `finish_reason` | FinishReason? | | |
| `usage` | Usage? | | |
| `error` | ErrorInfo? | | required if type = "error" |

---

## Live Sessions

### LiveConfig

| Field | Type | Required | Constraints |
|---|---|---|---|
| `model` | string | ✅ | non-empty |
| `system` | string \| Part[]? | | if Part[], non-empty |
| `tools` | Tool[] | `[]` | |
| `voice` | string? | | |
| `input_format` | AudioFormat? | | |
| `output_format` | AudioFormat? | | |
| `provider` | JsonObject? | | |

### LiveClientEvent

| Field | Type | Required | Constraints |
|---|---|---|---|
| `type` | `"audio"` \| `"video"` \| `"text"` \| `"tool_result"` \| `"interrupt"` \| `"end_audio"` | ✅ | |
| `data` | string? | | required if type ∈ {"audio", "video"} |
| `text` | string? | | required if type = "text" |
| `id` | string? | | required if type = "tool_result" |
| `content` | Part[] | `[]` | required if type = "tool_result" |

### LiveServerEvent

| Field | Type | Required | Constraints |
|---|---|---|---|
| `type` | `"audio"` \| `"text"` \| `"tool_call"` \| `"interrupted"` \| `"turn_end"` \| `"error"` | ✅ | |
| `data` | string? | | required if type = "audio" |
| `text` | string? | | required if type = "text" |
| `id` | string? | | required if type = "tool_call" |
| `name` | string? | | required if type = "tool_call" |
| `input` | JsonObject? | | required if type = "tool_call" |
| `usage` | Usage? | | required if type = "turn_end" |
| `error` | ErrorInfo? | | required if type = "error" |

---

## Auxiliary Request/Response Types

### EmbeddingRequest / EmbeddingResponse

```
EmbeddingRequest { model: string, inputs: string[], provider?: JsonObject }
EmbeddingResponse { model: string, vectors: float[][], usage?: Usage, provider?: JsonObject }
```

### FileUploadRequest / FileUploadResponse

```
FileUploadRequest { model?: string, filename: string, bytes_data: bytes, media_type: string, provider?: JsonObject }
FileUploadResponse { id: string, provider?: JsonObject }
```

### BatchRequest / BatchResponse

```
BatchRequest { model: string, requests: LMRequest[], provider?: JsonObject }
BatchResponse { id: string, status: string, provider?: JsonObject }
```

### ImageGenerationRequest / ImageGenerationResponse

```
ImageGenerationRequest { model: string, prompt: string, size?: string, provider?: JsonObject }
ImageGenerationResponse { images: DataSource[], provider?: JsonObject }
```

### AudioGenerationRequest / AudioGenerationResponse

```
AudioGenerationRequest { model: string, prompt: string, voice?: string, format?: string, provider?: JsonObject }
AudioGenerationResponse { audio: DataSource, provider?: JsonObject }
```

---

## Serialization

All types must be round-trippable through JSON via `to_dict` / `from_dict` functions. The canonical JSON representation omits null/empty fields. See `fixtures/` for examples.

---

## Error Hierarchy

```
ULMError
└── TransportError
└── ProviderError
    ├── AuthError              (401, 403)
    ├── BillingError           (402)
    ├── RateLimitError         (429)
    ├── InvalidRequestError    (400, 404, 409, 413, 422)
    │   └── ContextLengthError
    ├── TimeoutError           (408, 504)
    ├── ServerError            (5xx)
    ├── UnsupportedModelError
    ├── UnsupportedFeatureError
    └── NotConfiguredError
```

Each error class maps to a canonical `ErrorCode` string. Implementations must provide `map_http_error(status, message) → ProviderError`.

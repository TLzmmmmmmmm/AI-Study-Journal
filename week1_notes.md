# AI Application Fundamentals

> Core notes for building LLM applications.  
> Transformer architecture is documented separately in `Transformer_Notes_Refined.md`.

---

# 1. LLM Inference

## Context Window

The **context window** is the maximum amount of token information a model can process in one request.

It may contain:

- system instructions
- conversation history
- retrieved documents
- current user message
- generated output

All of them consume context tokens.

A longer context usually means higher:

- memory usage
- inference cost
- latency

Conversation history therefore cannot grow indefinitely.

---

## KV Cache

During autoregressive generation, previous attention Keys and Values can be reused.

**KV Cache** stores them instead of recomputing them for every new token.

Main benefit:

- faster generation

Trade-off:

- consumes additional memory

KV Cache mainly improves **inference speed**, not model intelligence.

---

## Temperature

`temperature` controls randomness during token sampling.

Low temperature:

- more stable
- more predictable
- more conservative

High temperature:

- more varied
- more creative
- less predictable

Important:

`higher temperature ≠ higher intelligence`

Typical use:

- factual QA / customer support / RAG: lower randomness
- brainstorming / creative writing: higher randomness

---

## Thinking Mode

Some models support an explicit reasoning mode.

Thinking mode is more useful for:

- multi-step reasoning
- difficult coding
- planning
- complex recommendations

It is usually unnecessary for:

- contact information
- simple FAQ
- direct product specifications

Thinking generally increases:

- latency
- token usage
- cost

---

## reasoning_effort

`reasoning_effort` controls how much reasoning effort the model uses when reasoning is enabled.

It is different from temperature:

- `temperature`: randomness
- `reasoning_effort`: reasoning depth/effort

Do not use maximum reasoning for every request.

---

# 2. LLM API Fundamentals

## SDK vs API Provider vs Model

These are different concepts.

Example:

- SDK: OpenAI Python SDK
- API provider: DeepSeek
- model: a DeepSeek model

Therefore:

`SDK ≠ API Provider ≠ Model`

An SDK is simply a convenient programming interface for making API requests.

---

## API Key

An API Key is a secret credential used for:

- authentication
- authorization
- usage tracking
- billing

Never:

- put it in frontend JavaScript
- commit it to GitHub
- hardcode it into public source code

API Keys belong on the backend.

---

## Model

The `model` parameter selects which model handles the request.

Models may differ in:

- capability
- latency
- price
- reasoning ability
- context length

The strongest model does not need to handle every request.

---

## Messages

Chat APIs usually represent conversation input as messages.

Common roles:

- `system`
- `user`
- `assistant`
- `tool`

Example:

```json
[
  {
    "role": "system",
    "content": "You are a customer support assistant."
  },
  {
    "role": "user",
    "content": "What products do you offer?"
  }
]
```

---

## System Prompt vs User Prompt

**System Prompt**

Defines how the model should behave.

Examples:

- role
- response style
- constraints
- safety rules

**User Prompt**

Defines what the user wants in the current request.

Simple distinction:

- System Prompt: how to answer
- User Prompt: what to answer

---

## Prompt Reliability

Prompts are instructions, not deterministic program logic.

Models can still:

- misunderstand instructions
- ignore constraints
- hallucinate
- return unexpected formats

Production systems therefore also use:

- validation
- structured output
- RAG
- deterministic rules
- error handling
- evaluation

---

## API Response

An LLM API response may contain:

- generated content
- model information
- token usage
- metadata
- finish information

For Chat Completions:

```python
response.choices[0].message.content
```

extracts the assistant text.

---

# 3. Structured Output

Natural-language output is easy for humans but harder for programs to use reliably.

Instead of:

```text
I think this is a product question.
```

a program may prefer:

```json
{
  "category": "product",
  "confidence": 0.94
}
```

---

## Schema

A schema defines expected:

- fields
- types
- allowed values
- required values

Example:

```text
category: product | solution | support | other
confidence: number
needs_rag: boolean
```

---

## Prompt vs JSON Mode vs Structured Output

**Prompt**

"Please return JSON."

This is only a soft instruction.

**JSON Mode**

Guarantees valid JSON syntax.

It does not necessarily guarantee the correct fields.

**Structured Output**

Constrains output using a schema.

It improves:

- syntax reliability
- field reliability
- machine readability

---

## Important Limitation

Structured Output does **not** guarantee factual correctness.

A model can produce perfectly valid JSON containing incorrect information.

Therefore:

`Structured Output ≠ anti-hallucination`

---

## Confidence

A model output such as:

```json
{
  "confidence": 0.94
}
```

does not automatically mean there is a mathematically calibrated 94% probability of correctness.

LLM confidence usually needs evaluation and calibration before being trusted for important decisions.

---

# 4. Routing

A production AI application does not need to send every request through the same processing path.

A router can classify requests based on:

- intent
- complexity
- ambiguity
- retrieval requirements
- reasoning requirements
- risk

---

## Rule-Based Routing

Advantages:

- fast
- cheap
- deterministic
- easy to debug

Disadvantages:

- weak natural-language understanding
- difficult to maintain when rules become numerous

Use rules for highly predictable cases.

---

## LLM Routing

A small LLM can classify a request.

Advantages:

- flexible
- understands natural language

Disadvantages:

- additional API call
- additional latency
- additional cost
- classification can still be wrong

---

## Hybrid Routing

A useful long-term approach:

1. Check high-confidence deterministic rules.
2. If rules cannot classify reliably, use an LLM router.

Do not attempt to encode every possible natural-language request using rules.

---

## No-LLM Path

Some requests do not require an LLM.

Example:

> What is your company phone number?

If the value already exists in structured data, ordinary code can return it.

Benefits:

- deterministic
- fast
- almost zero token cost

Principle:

**Do not use an LLM when ordinary software is sufficient.**

---

# 5. Why Not LangChain Yet?

LangChain can abstract:

- model calls
- prompts
- retrieval
- tools
- agents

But abstractions are easier to understand after learning the underlying components.

Recommended learning order:

1. direct model API
2. HTTP
3. FastAPI
4. structured output
5. RAG
6. tool calling
7. routing / agents
8. frameworks when complexity justifies them

Principle:

**Understand the abstraction before depending on it.**

---

# 6. HTTP Fundamentals

## HTTP Request

An HTTP request contains four important parts:

- Method
- URL
- Headers
- Body

Questions:

- Method: what operation?
- URL: where?
- Headers: how should the request be interpreted?
- Body: what data is being sent?

---

## GET

`GET` is mainly used to retrieve information.

Examples:

```text
GET /health
GET /products
```

GET requests should normally not modify server state.

---

## POST

`POST` is commonly used to submit data for processing.

Examples:

```text
POST /api/chat
POST /orders
```

AI chat naturally uses POST because the client sends user input to the server for processing.

---

## Headers

Headers contain metadata about a request or response.

Examples:

```http
Content-Type: application/json
Authorization: Bearer API_KEY
```

`Content-Type` describes the format of the body.

---

## application/json

Media types use:

```text
type/subtype
```

For:

```text
application/json
```

- `application` is the broad media category
- `json` is the specific data format

It tells the receiver to interpret the body as JSON.

---

## JSON

JSON is a structured data-exchange format.

Common JSON types:

- string
- number
- boolean
- null
- array
- object

Example:

```json
{
  "message": "Hello",
  "stream": false
}
```

---

## HTTP Response

An HTTP response contains:

- status code
- headers
- body

---

## Important Status Codes

| Code | Meaning |
|---|---|
| 200 | Success |
| 400 | Bad Request |
| 401 | Authentication Failed |
| 403 | Forbidden |
| 404 | Not Found |
| 422 | Validation Failed |
| 429 | Rate Limited |
| 500 | Internal Server Error |
| 502 | Upstream Service Error |
| 503 | Service Unavailable |
| 504 | Upstream Timeout |

General categories:

- `2xx`: success
- `4xx`: client/request problem
- `5xx`: server/service problem

---

# 7. FastAPI Fundamentals

## FastAPI App

```python
app = FastAPI()
```

creates the FastAPI application.

---

## Route

A route connects an HTTP method and path to a Python function.

```python
@app.get("/health")
def health():
    return {"status": "ok"}
```

---

## Why `/api/chat`?

`/api/chat` is a naming convention, not a FastAPI requirement.

Using `/api/...` helps distinguish API endpoints from website pages.

For example:

```text
/products
/about
/api/chat
/api/products
```

An API-only domain such as `api.example.com` may not need the extra `/api` prefix.

---

## Pydantic

Pydantic defines and validates request data.

```python
class ChatRequest(BaseModel):
    message: str
```

FastAPI can automatically:

- parse JSON
- validate required fields
- validate data types
- return validation errors

After validation:

```python
request.message
```

contains the user message.

---

## `async`

`async` is useful when code spends time waiting for I/O such as:

- HTTP requests
- database queries
- network operations

Important:

`async ≠ multithreading`

Do not use `async def` simply because FastAPI supports it.

Use asynchronous code when the underlying operation/library supports `await`.

---

## Exception Handling

Python uses:

```python
try:
    ...
except SomeException:
    ...
```

FastAPI can convert failures into HTTP errors:

```python
raise HTTPException(
    status_code=503,
    detail="AI service unavailable"
)
```

Typical mappings:

- upstream timeout: `504`
- connection failure: `503`
- bad upstream response: `502`
- unexpected backend error: `500`

Specific exceptions should be handled before generic `Exception`.

---

## Fail Fast

Critical server configuration should be checked during startup.

Example:

```python
api_key = os.environ.get("DEEPSEEK_API_KEY")

if not api_key:
    raise RuntimeError("DEEPSEEK_API_KEY is not set")
```

It is better for a misconfigured server to fail during startup than fail only after users send requests.

---

# 8. Secrets & Environment Variables

## `.env`

Example:

```env
DEEPSEEK_API_KEY=...
```

`.env` is only a text configuration file.

It does not load itself.

---

## `load_dotenv()`

```python
from dotenv import load_dotenv

load_dotenv()
```

loads values from `.env` into the Python process environment.

Then:

```python
os.environ.get("DEEPSEEK_API_KEY")
```

can access them.

---

## `.gitignore`

Typical entries:

```gitignore
.env
.venv/
__pycache__/
*.py[cod]
```

`.gitignore` does not protect a secret that has already been committed.

If an API Key was published:

**revoke/rotate it immediately.**

Deleting it from the latest commit is not enough.

---

## Frontend Secrets

Anything delivered to a browser should be considered public.

Users can inspect:

- HTML
- JavaScript
- frontend bundles
- HTTP requests
- request headers

Therefore the browser should call your backend, while the backend holds the external AI API Key.

---

# 9. Frontend HTTP

## `fetch()`

`fetch()` is the browser's native HTTP request API.

Example:

```javascript
const response = await fetch("/api/chat", {
    method: "POST",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify({
        message: input.value
    })
});
```

Without specifying a method, `fetch()` defaults to GET.

---

## `JSON.stringify()`

JavaScript objects are converted into JSON text before being sent.

```javascript
JSON.stringify({
    message: "Hello"
})
```

produces:

```json
{"message":"Hello"}
```

---

## `response.json()`

```javascript
const data = await response.json();
```

parses a JSON response body into a JavaScript object.

Then:

```javascript
data.answer
```

can access the `answer` field.

---

# 10. JavaScript Async / Await

`fetch()` returns a **Promise**.

A Promise represents a result that may become available later.

```javascript
const response = await fetch(...)
```

waits for the Promise to resolve before continuing that function.

`await` is normally used inside an `async` function.

Important:

`await` does not mean the entire browser freezes while waiting.

---

# 11. Frontend State

An AI chat interface commonly has four conceptual states:

- `idle`
- `loading`
- `success`
- `error`

---

## idle

`idle` means the application is waiting for user action.

It is a conceptual state, not a JavaScript keyword.

---

## loading state

Used while waiting for the backend.

Typical UI behavior:

- show "AI is thinking..."
- disable the Send button
- show a loading indicator

This gives feedback and prevents duplicate requests.

---

## error state

Used when a request fails.

The UI should display a useful error rather than silently failing.

Example state:

```javascript
loading = false;
error = "AI service unavailable";
```

---

# 12. `response.ok`

A critical `fetch()` behavior:

HTTP errors such as `404`, `500`, or `503` usually do **not** automatically cause `fetch()` to throw.

Check:

```javascript
if (!response.ok) {
    throw new Error("Request failed");
}
```

Approximately:

- status `200–299`: `response.ok === true`
- otherwise: `false`

---

# 13. `throw` vs `raise`

JavaScript:

```javascript
throw new Error("Request failed");
```

Python:

```python
raise Exception("Request failed")
```

They serve the same general purpose:

**interrupt normal execution and trigger exception handling.**

| JavaScript | Python |
|---|---|
| `throw` | `raise` |
| `Error` | `Exception` |
| `catch` | `except` |

A frontend may convert an HTTP failure into a JavaScript exception using `throw`.

A backend may convert a Python exception into an HTTP error using `raise HTTPException`.

---

# 14. DOM

DOM = **Document Object Model**.

The browser represents HTML elements as objects that JavaScript can access.

Example:

```javascript
const input = document.getElementById("message");
```

To update displayed content:

```javascript
answerElement.textContent = data.answer;
```

Direct DOM manipulation is suitable for small interfaces.

---

# 15. State Management

Application state represents what the UI currently knows.

Common chat state:

```javascript
input
messages
loading
error
```

Examples:

- `messages`: conversation history
- `loading`: whether a request is running
- `error`: current error information
- `input`: current user input

For simple JavaScript applications, state and DOM can be managed manually.

For larger React/Vue/Svelte applications, the preferred idea is:

**change application state and let the framework update the DOM.**

---

# 16. Core Principles

1. `SDK ≠ API provider ≠ model`.
2. Prompts are instructions, not hard guarantees.
3. Structured Output guarantees structure, not factual correctness.
4. Do not use an LLM when deterministic software is sufficient.
5. Do not use maximum reasoning/model capability for every request.
6. API Keys must stay on the backend.
7. HTTP APIs define contracts between frontend and backend.
8. Validate external data before passing it into core logic.
9. Convert backend failures into meaningful HTTP status codes.
10. `fetch()` does not automatically throw on normal HTTP error responses.
11. Frontends must explicitly handle loading and error states.
12. UI complexity is easier to manage when it is driven by state.
13. Learn the underlying components before introducing frameworks such as LangChain.

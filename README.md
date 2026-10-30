# 🚀 Production LLM Serving Platform

A portfolio project demonstrating a FastAPI gateway for Large Language Model (LLM) applications. It includes provider abstraction, fallback routing, response caching, rate limiting, health checks, request tracking, and basic operational metrics.

> **Important:** This is an educational starter project, not a hardened production service. It defaults to a mock provider, so you can explore the implementation without an API key or paid inference calls.

## ✨ Features

- ⚡ FastAPI-based LLM serving gateway
- 🤖 Mock LLM provider for local development
- 🔌 Optional OpenAI-compatible API integration
- 🔄 Provider fallback mechanism
- 🚀 In-memory response caching
- 🛡️ Sliding-window rate limiting
- 🩺 Health-check endpoint
- 📊 Request and latency metrics
- 🆔 Request ID tracking
- 🧪 API smoke tests
- 📓 Interactive Jupyter Notebook walkthrough

## 🏗️ Architecture

```text
                  Client Application
                         |
                         v
                  FastAPI Gateway
                         |
              +----------+----------+
              |          |          |
              v          v          v
          Validation   Rate Limit   Request ID
              |          |          |
              +----------+----------+
                         |
                         v
                   Response Cache
                         |
                  Cache Miss?
                         |
                         v
                   Provider Router
                         |
                +--------+--------+
                |                 |
                v                 v
          Primary Provider   Fallback Provider
                |                 |
                +--------+--------+
                         |
                         v
                 LLM Response
                         |
                         v
                 Metrics Tracking
                         |
                         v
                    API Client
```

## 🛠️ Technology Stack

| Technology | Purpose |
|---|---|
| Python | Core programming language |
| FastAPI | API development |
| Pydantic | Request validation |
| HTTPX | HTTP communication with compatible model APIs |
| Jupyter Notebook | Interactive development and testing |
| Uvicorn | ASGI application server |
| Pytest | Testing support |

## 📁 Project Structure

```text
production-llm-serving-platform/
│
├── production_llm_serving_platform.ipynb
├── README.md
├── requirements.txt
├── .env.example
└── .gitignore
```

The notebook includes a cell that exports the FastAPI implementation to `app.py` so you can run it as a standalone API.

## ⚙️ Prerequisites

- Python 3.10 or later
- Jupyter Notebook or Google Colab
- Basic knowledge of Python and REST APIs
- An API key only if you choose to configure an external LLM provider

## 🚀 Getting Started

### 1. Clone the repository

Replace `YOUR_USERNAME` with your GitHub username.

```bash
git clone https://github.com/YOUR_USERNAME/production-llm-serving-platform.git
```

### 2. Navigate to the project directory

```bash
cd production-llm-serving-platform
```

### 3. Create a virtual environment

**Windows PowerShell**

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

**macOS/Linux**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 4. Install dependencies

```bash
pip install -r requirements.txt
```

### 5. Launch Jupyter Notebook

```bash
jupyter notebook production_llm_serving_platform.ipynb
```

Run the notebook cells from top to bottom.

The default mock provider does not require an API key.

## 🧠 How It Works

### 1. Request Validation

The gateway validates incoming requests using Pydantic models.

It checks:

- Message roles and content
- Request message count
- Temperature settings
- Maximum token configuration

Invalid requests are rejected before reaching the provider.

### 2. Provider Abstraction

The project defines a common interface for model providers.

The default mock provider returns a deterministic response, allowing you to test the serving architecture without calling an external model.

An optional OpenAI-compatible provider demonstrates how to forward requests to a compatible chat-completions API.

### 3. Response Caching

The gateway creates a hash from the request payload and checks its in-memory cache.

When a matching, unexpired entry exists, the gateway returns the cached response instead of calling the provider again.

Benefits include:

- Reduced repeated provider calls
- Lower latency for repeated requests
- Potential reduction in inference costs

The example cache is process-local and is not shared between multiple server workers.

### 4. Rate Limiting

A sliding-window rate limiter tracks requests by client IP address.

When the configured request limit is exceeded, the gateway returns HTTP `429 Too Many Requests`.

For distributed production deployments, replace this in-memory implementation with a shared solution such as Redis.

### 5. Provider Fallback

The gateway attempts to use the primary provider first.

If that provider raises an exception, the gateway attempts to generate a response using the fallback provider.

The example uses the mock provider as its fallback. This demonstrates the routing pattern, but it does not guarantee that the fallback response will have the same quality or capabilities as the primary model.

### 6. Monitoring and Metrics

The metrics endpoint exposes basic information about requests processed by the current application process.

Metrics include:

- Total requests
- Error count
- Error rate
- Cache hits
- Provider fallback count
- Average latency
- Approximate 95th-percentile latency

These metrics are useful for demonstrating observability concepts. They are not a replacement for a production monitoring system.

## 🌐 API Endpoints

After exporting `app.py` from the notebook, start the API using:

```bash
uvicorn app:app --reload
```

The default local API address is:

```text
http://127.0.0.1:8000
```

### Available Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/health` | Returns application health and provider information |
| GET | `/metrics` | Returns basic request and latency metrics |
| POST | `/v1/chat/completions` | Generates a response through the configured provider |
| GET | `/docs` | Opens the interactive Swagger API documentation |

### Example API Request

**Endpoint**

```text
POST /v1/chat/completions
```

**Request body**

```json
{
  "messages": [
    {
      "role": "user",
      "content": "Explain LLM serving in one sentence."
    }
  ],
  "temperature": 0.2,
  "max_tokens": 100
}
```

### Example Response

The mock provider returns a response similar to:

```json
{
  "id": "example-request-id",
  "model": "gpt-4o-mini",
  "content": "[Mock LLM response] You asked: Explain LLM serving in one sentence.",
  "provider": "mock",
  "cached": false,
  "latency_ms": 1.25
}
```

The response ID and latency are generated at runtime, so actual values will vary.

## 🔐 Optional LLM Provider Configuration

To use an external OpenAI-compatible provider, configure these environment variables:

```env
LLM_PROVIDER=openai_compatible
LLM_API_BASE=https://api.openai.com/v1
LLM_API_KEY=your_api_key
LLM_MODEL=gpt-4o-mini
```

The model name and API base URL are examples. Choose values supported by your provider.

You can also configure the cache and rate limiter:

```env
CACHE_TTL_SECONDS=60
RATE_LIMIT_REQUESTS=30
RATE_LIMIT_WINDOW_SECONDS=60
```

The notebook reads environment variables when it is initialized. If you change them after initialization, restart the kernel and run the relevant cells again.

**Security:** Never commit real API keys or your `.env` file to GitHub. Use environment variables or a secrets manager.

## 🧪 Testing

The notebook includes basic smoke tests that demonstrate:

- The health endpoint returns a successful response
- The chat endpoint accepts a valid request
- Repeated identical requests can use the cache
- Invalid message roles are rejected
- Request IDs are returned in response headers
- Metrics can be retrieved from the API

These are starter-level tests. Add dedicated automated tests for provider failures, rate-limit boundaries, cache expiry, concurrency, and other production requirements.

## 📈 Production-Hardening Roadmap

Before exposing this application to real users, consider implementing:

- Authentication and authorization
- Per-user or per-tenant quotas
- Redis-backed caching and distributed rate limiting
- Provider-specific retries and bounded timeouts
- Circuit breakers and robust error mapping
- Streaming responses where required
- Persistent metrics and dashboards
- Structured logs and distributed tracing
- Load testing and concurrency controls
- PII handling and data-retention policies
- Prompt-injection and abuse protections
- Secrets management and key rotation
- Automated security and dependency checks
- Containerization and deployment automation

The current implementation uses in-memory state, which resets when the process restarts and is not shared across multiple workers or instances. It should not be treated as production-ready without these additional controls and testing.

## ☁️ Deployment Ideas

A practical next step is to deploy the API backend to Render.

If you later build a frontend, you can deploy it separately to Vercel.

For a larger deployment, investigate managed caching, centralized metrics, tracing, load balancing, and autoscaling.

## 🎯 Learning Outcomes

By completing this project, you will practice:

- Building an LLM gateway with FastAPI
- Designing a provider abstraction
- Implementing fallback routing
- Understanding response caching
- Applying basic rate limiting
- Exposing operational metrics
- Testing REST API behavior
- Identifying production reliability and security requirements

## 🔮 Future Enhancements

- Redis-based distributed cache
- Prometheus-compatible metrics
- OpenTelemetry tracing
- Provider health checks and intelligent routing
- Model benchmarking and cost comparison
- Streaming chat responses
- Load-testing scripts
- Docker-based deployment
- Continuous integration and automated tests
- Frontend dashboard for monitoring requests

## 📄 License

Choose a license before publishing this repository publicly. MIT is one option for an open-source portfolio project.

---

**Built as a Generative AI engineering portfolio project.**

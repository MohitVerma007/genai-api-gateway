# 🚀 Multi-Provider GenAI Integration API Gateway

> Production-scale, asynchronous API Gateway for unified LLM provider management with intelligent routing, rate-limiting, and fault tolerance.

[![Python](https://img.shields.io/badge/Python-3.9+-3776ab?logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Azure](https://img.shields.io/badge/Azure-Cloud-0078D4?logo=microsoft-azure&logoColor=white)](https://azure.microsoft.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Tech Stack](#tech-stack)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [API Endpoints](#api-endpoints)
- [Monitoring & Observability](#monitoring--observability)
- [Contributing](#contributing)
- [License](#license)

---

## 🎯 Overview

The **GenAI API Gateway** is a sophisticated request orchestration layer designed to abstract the complexity of managing multiple Large Language Model (LLM) providers. It provides:

- **Unified Interface**: Single API endpoint for all LLM interactions regardless of backend provider
- **Provider Agnostic**: Seamless switching between Azure OpenAI, HuggingFace, and future providers
- **High Availability**: 99.9% uptime with intelligent fallback mechanisms and provider health monitoring
- **Real-time Streaming**: Asynchronous request handling with Server-Sent Events (SSE) for live response streaming
- **Enterprise Ready**: Rate limiting, authentication, request validation, and comprehensive logging

### Problem Statement

Organizations using multiple LLM providers face several challenges:
- **API Fragmentation**: Each provider has different request/response schemas
- **Vendor Lock-in**: Switching providers requires extensive code refactoring
- **Reliability**: Single provider outage impacts entire application
- **Cost Optimization**: No automatic routing to cheapest available provider
- **Monitoring Complexity**: Tracking metrics across multiple providers is cumbersome

### Solution

Our gateway standardizes interactions through a **canonical data contract**, enabling:
- Plug-and-play provider switching
- Transparent failover during outages
- Cost-aware routing algorithms
- Unified observability and metrics

---

## ✨ Key Features

### 1. **Intelligent Request Routing**
   - Automatic load distribution across healthy providers
   - Cost-optimized routing based on per-token pricing
   - Geographic routing for latency minimization
   - Custom routing policies via metadata flags

### 2. **Provider Health Monitoring**
   - Real-time health check probes (configurable intervals)
   - Graceful degradation during provider outages
   - Automatic recovery detection and circuit breaker pattern
   - Health status dashboard

### 3. **Rate Limiting & Quota Management**
   - Token-bucket algorithm for fair resource allocation
   - Per-user, per-model, and global rate limits
   - Quota tracking and enforcement
   - Burst allowance configuration

### 4. **Standardized Data Contract**
   - Universal request/response schema across all providers
   - Automatic mapping to provider-specific formats
   - Type validation using Pydantic v2
   - Support for streaming and non-streaming modes

### 5. **Real-time Response Streaming**
   - Server-Sent Events (SSE) for live token streaming
   - Graceful error handling in streams
   - Automatic reconnection support
   - Backpressure management

### 6. **Fallback Mechanism**
   - Primary → Secondary → Tertiary provider chain
   - Configurable fallback policies
   - Retry logic with exponential backoff
   - Provider-specific error handling

### 7. **Comprehensive Observability**
   - Structured logging with correlation IDs
   - Prometheus metrics export
   - Distributed tracing support (OpenTelemetry ready)
   - Request/response audit trails

### 8. **Security & Authentication**
   - OAuth 2.0 / API Key authentication
   - Request signing and validation
   - CORS configuration
   - Rate limiting per user/org

---

## 🏗️ Architecture

### High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                      Client Applications                         │
└────────────┬────────────────────────────┬────────────────────────┘
             │                            │
             └────────────┬───────────────┘
                          │
        ┌─────────────────▼──────────────────┐
        │   API Gateway (FastAPI)            │
        │  - Request Validation              │
        │  - Authentication/Authorization    │
        │  - Rate Limiting                   │
        └─────────────────┬──────────────────┘
                          │
        ┌─────────────────▼──────────────────┐
        │   Request Router                   │
        │  - Provider Selection Logic        │
        │  - Health Check Integration        │
        │  - Cost Optimization               │
        └──┬──────────────┬────────────┬─────┘
           │              │            │
    ┌──────▼──┐    ┌──────▼──┐    ┌──▼───────┐
    │ Azure   │    │Hugging  │    │ Future   │
    │ OpenAI  │    │ Face    │    │Providers │
    │Provider │    │Provider │    │ Adapter  │
    └──────┬──┘    └──────┬──┘    └──┬───────┘
           │              │          │
    ┌──────▼──────────────▼──────────▼──┐
    │   Unified Response Mapper          │
    │  - Schema Translation              │
    │  - Streaming Management            │
    └──────┬───────────────────────────┬─┘
           │                           │
        ┌──▼──────────┐         ┌──────▼──┐
        │  SSE Stream │         │  Cache  │
        │   Handler   │         │ (Redis) │
        └─────────────┘         └─────────┘

   ┌─────────────────────────────────────────┐
   │   Observability Layer                   │
   │  - Prometheus Metrics                   │
   │  - Structured Logging                   │
   │  - Distributed Tracing                  │
   │  - Health Dashboards                    │
   └─────────────────────────────────────────┘
```

### Data Flow

1. **Ingestion**: Client sends request to unified API endpoint
2. **Validation**: Request schema validated against canonical contract
3. **Authentication**: API key/token verification
4. **Rate Checking**: Verify user hasn't exceeded quotas
5. **Routing**: Select optimal provider based on:
   - Provider health status
   - Current load
   - Cost metrics
   - User preferences
6. **Transformation**: Map canonical request to provider-specific format
7. **Execution**: Call selected provider's API
8. **Transformation Back**: Map provider response to canonical schema
9. **Streaming**: Send response via SSE or return full response
10. **Metrics**: Record latency, tokens, cost, status

---

## 📁 Project Structure

```
genai-api-gateway/
├── README.md                          # Project documentation
├── LICENSE                             # MIT License
├── .gitignore                          # Git ignore patterns
├── .env.example                        # Environment variables template
├── pyproject.toml                      # Poetry/pip dependencies
├── poetry.lock                         # Locked dependencies (Poetry)
├── requirements.txt                    # pip requirements
│
├── src/
│   └── gateway/
│       ├── __init__.py
│       ├── main.py                     # FastAPI app initialization
│       ├── config.py                   # Configuration management
│       ├── logging_config.py           # Structured logging setup
│       │
│       ├── api/
│       │   ├── __init__.py
│       │   ├── routes.py               # API endpoints
│       │   ├── dependencies.py         # FastAPI dependencies (auth, etc)
│       │   └── exceptions.py           # Custom exception handlers
│       │
│       ├── schemas/
│       │   ├── __init__.py
│       │   ├── request.py              # Request data models
│       │   ├── response.py             # Response data models
│       │   └── models.py               # Shared models
│       │
│       ├── providers/
│       │   ├── __init__.py
│       │   ├── base.py                 # Abstract provider interface
│       │   ├── azure_openai.py         # Azure OpenAI implementation
│       │   ├── huggingface.py          # HuggingFace implementation
│       │   ├── provider_factory.py     # Provider instantiation
│       │   └── adapter.py              # Schema mapping logic
│       │
│       ├── router/
│       │   ├── __init__.py
│       │   ├── router.py               # Main routing logic
│       │   ├── health_check.py         # Provider health monitoring
│       │   ├── cost_calculator.py      # Cost-based routing
│       │   └── policies.py             # Routing policy definitions
│       │
│       ├── rate_limiter/
│       │   ├── __init__.py
│       │   ├── limiter.py              # Token bucket algorithm
│       │   ├── storage.py              # Rate limit state storage
│       │   └── decorators.py           # Rate limit decorators
│       │
│       ├── cache/
│       │   ├── __init__.py
│       │   ├── redis_cache.py          # Redis cache implementation
│       │   └── cache_manager.py        # Cache lifecycle management
│       │
│       ├── streaming/
│       │   ├── __init__.py
│       │   ├── sse_handler.py          # Server-Sent Events handler
│       │   └── stream_utils.py         # Streaming utilities
│       │
│       ├── monitoring/
│       │   ├── __init__.py
│       │   ├── metrics.py              # Prometheus metrics
│       │   ├── tracer.py               # OpenTelemetry tracing
│       │   └── logger.py               # Structured logging
│       │
│       └── utils/
│           ├── __init__.py
│           ├── helpers.py              # Utility functions
│           └── constants.py            # Application constants
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py                     # Pytest fixtures
│   ├── test_config.py
│   │
│   ├── unit/
│   │   ├── __init__.py
│   │   ├── test_routing.py
│   │   ├── test_rate_limiter.py
│   │   ├── test_providers.py
│   │   └── test_schemas.py
│   │
│   ├── integration/
│   │   ├── __init__.py
│   │   ├── test_api_endpoints.py
│   │   ├── test_azure_provider.py
│   │   └── test_huggingface_provider.py
│   │
│   └── performance/
│       ├── __init__.py
│       └── test_load.py                # Load testing
│
├── docker/
│   ├── Dockerfile                      # Production Docker image
│   ├── Dockerfile.dev                  # Development Docker image
│   └── docker-compose.yml              # Local development setup
│
├── kubernetes/
│   ├── deployment.yaml                 # K8s deployment config
│   ├── service.yaml                    # K8s service config
│   ├── hpa.yaml                        # Horizontal Pod Autoscaler
│   └── configmap.yaml                  # K8s ConfigMap
│
├── docs/
│   ├── API.md                          # API documentation
│   ├── ARCHITECTURE.md                 # Architecture deep-dive
│   ├── PROVIDERS.md                    # Provider integration guide
│   ├── DEPLOYMENT.md                   # Deployment instructions
│   ├── MONITORING.md                   # Monitoring & alerting
│   └── TROUBLESHOOTING.md              # Troubleshooting guide
│
├── examples/
│   ├── basic_usage.py                  # Basic client example
│   ├── streaming_example.py            # Streaming response example
│   ├── custom_routing.py               # Custom routing policy example
│   └── client.py                       # Python SDK client
│
├── scripts/
│   ├── setup.sh                        # Initial setup script
│   ├── health_check.sh                 # Health check script
│   └── load_test.sh                    # Load testing script
│
├── .github/
│   ├── workflows/
│   │   ├── ci.yml                      # CI pipeline
│   │   ├── tests.yml                   # Test pipeline
│   │   └── deploy.yml                  # Deploy pipeline
│   └── CONTRIBUTING.md
│
└── .env.example                        # Environment template
```

---

## 🛠️ Tech Stack

### Core Framework
- **FastAPI** - Modern, fast web framework for building APIs
- **Pydantic v2** - Data validation and parsing
- **Python 3.9+** - Programming language

### Provider SDKs
- **azure-openai** - Azure OpenAI Service SDK
- **huggingface-hub** - HuggingFace Hub integration

### Async & Concurrency
- **asyncio** - Asynchronous I/O
- **aiohttp** - Async HTTP client
- **httpx** - HTTP client with async support

### Caching & Storage
- **redis** - In-memory data store for caching and rate limiting
- **aioredis** - Async Redis client

### Monitoring & Observability
- **prometheus-client** - Prometheus metrics
- **python-json-logger** - JSON structured logging
- **opentelemetry** - Distributed tracing
- **pythonjsonlogger** - JSON logging

### Testing
- **pytest** - Testing framework
- **pytest-asyncio** - Async test support
- **pytest-cov** - Code coverage
- **httpx** - Test client

### DevOps & Deployment
- **Docker** - Containerization
- **Kubernetes** - Orchestration
- **Uvicorn** - ASGI server

### Development
- **Poetry** - Dependency management
- **Black** - Code formatter
- **Ruff** - Fast Python linter
- **mypy** - Type checking
- **pre-commit** - Git hooks

---

## 🚀 Installation

### Prerequisites
- Python 3.9 or higher
- Docker & Docker Compose (optional, for containerized setup)
- Redis (for caching/rate limiting)
- Azure OpenAI API key
- HuggingFace API token

### Local Setup (Development)

#### 1. Clone Repository
```bash
git clone https://github.com/MohitVerma007/genai-api-gateway.git
cd genai-api-gateway
```

#### 2. Create Virtual Environment
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

#### 3. Install Dependencies
```bash
# Using Poetry (recommended)
pip install poetry
poetry install

# Or using pip
pip install -r requirements.txt
```

#### 4. Configure Environment
```bash
cp .env.example .env
# Edit .env with your credentials
```

#### 5. Run Redis (if not using Docker Compose)
```bash
# macOS with Homebrew
brew install redis
brew services start redis

# Or Docker
docker run -d -p 6379:6379 redis:latest
```

#### 6. Start the Application
```bash
# Development server with auto-reload
uvicorn src.gateway.main:app --reload --host 0.0.0.0 --port 8000

# Or using Poetry
poetry run uvicorn src.gateway.main:app --reload
```

#### 7. Access the Application
- **API**: http://localhost:8000
- **API Docs**: http://localhost:8000/docs (Swagger UI)
- **ReDoc**: http://localhost:8000/redoc

### Docker Setup

```bash
# Build and run with Docker Compose
docker-compose up -d

# View logs
docker-compose logs -f gateway

# Shutdown
docker-compose down
```

---

## ⚙️ Configuration

### Environment Variables

Create `.env` file based on `.env.example`:

```env
# Server Configuration
SERVER_HOST=0.0.0.0
SERVER_PORT=8000
ENVIRONMENT=development  # development, staging, production
DEBUG=True

# Azure OpenAI Configuration
AZURE_OPENAI_API_KEY=your_azure_api_key
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4

# HuggingFace Configuration
HUGGINGFACE_API_TOKEN=your_hf_token
HUGGINGFACE_MODEL_ID=meta-llama/Llama-2-7b

# Redis Configuration
REDIS_URL=redis://localhost:6379/0
REDIS_CACHE_TTL=3600

# Rate Limiting
RATE_LIMIT_ENABLED=True
RATE_LIMIT_REQUESTS_PER_MINUTE=100
RATE_LIMIT_TOKENS_PER_DAY=1000000

# Authentication
API_KEY_HEADER=X-API-Key
AUTH_ENABLED=True

# Monitoring
PROMETHEUS_ENABLED=True
TRACING_ENABLED=False
LOG_LEVEL=INFO

# Feature Flags
FEATURE_STREAMING=True
FEATURE_COST_OPTIMIZATION=True
FEATURE_HEALTH_CHECK=True
```

### Configuration File

Edit `src/gateway/config.py` to customize behavior:

```python
# Key configurations
MAX_RETRIES = 3
RETRY_BACKOFF_FACTOR = 1.5
HEALTH_CHECK_INTERVAL = 30  # seconds
ROUTING_POLICY = "cost-optimized"  # or "round-robin", "least-loaded"
STREAMING_CHUNK_SIZE = 1024  # bytes
```

---

## 📚 Usage

### Basic Example

```python
import asyncio
from src.gateway.schemas.request import CompletionRequest
from src.gateway.router.router import RequestRouter

async def main():
    # Initialize router
    router = RequestRouter()
    
    # Create request
    request = CompletionRequest(
        model="gpt-4",
        prompt="Explain quantum computing",
        max_tokens=500,
        temperature=0.7
    )
    
    # Get response
    response = await router.route_and_execute(request)
    print(response.choices[0].text)

if __name__ == "__main__":
    asyncio.run(main())
```

### Streaming Example

```python
import asyncio
from src.gateway.schemas.request import CompletionRequest
from src.gateway.router.router import RequestRouter

async def main():
    router = RequestRouter()
    
    request = CompletionRequest(
        model="gpt-4",
        prompt="Write a poem about AI",
        stream=True,
        max_tokens=300
    )
    
    # Stream response
    async for chunk in router.route_and_stream(request):
        print(chunk.choices[0].delta.content, end="", flush=True)

if __name__ == "__main__":
    asyncio.run(main())
```

### With Custom Routing Policy

```python
from src.gateway.router.policies import RoutingPolicy
from src.gateway.router.router import RequestRouter

class CustomRoutingPolicy(RoutingPolicy):
    async def select_provider(self, available_providers, request):
        # Custom logic: prefer cheaper provider with <50ms latency
        return min(
            available_providers,
            key=lambda p: p.cost_per_1k_tokens if p.latency < 50 else float('inf')
        )

# Use custom policy
router = RequestRouter(routing_policy=CustomRoutingPolicy())
```

---

## 🔌 API Endpoints

### Health & Status

```bash
# Health check
GET /health

# Provider status
GET /status/providers

# Metrics
GET /metrics
```

### Completions

```bash
# Create completion
POST /v1/completions
Content-Type: application/json
X-API-Key: your_api_key

{
  "model": "gpt-4",
  "prompt": "Hello, world!",
  "max_tokens": 100,
  "temperature": 0.7
}
```

### Chat Completions

```bash
# Create chat completion
POST /v1/chat/completions
Content-Type: application/json
X-API-Key: your_api_key

{
  "model": "gpt-4",
  "messages": [
    {"role": "user", "content": "Hello!"}
  ],
  "temperature": 0.7
}
```

### Streaming Response

```bash
# Streaming completion
POST /v1/completions
Content-Type: application/json
X-API-Key: your_api_key

{
  "model": "gpt-4",
  "prompt": "Tell a story",
  "stream": true,
  "max_tokens": 500
}

# Response: Server-Sent Events stream
data: {"choices":[{"delta":{"content":"Once"}}]}
data: {"choices":[{"delta":{"content":" upon"}}]}
...
```

---

## 📊 Monitoring & Observability

### Prometheus Metrics

Access metrics at `http://localhost:8000/metrics`:

```
# Request metrics
gateway_requests_total{provider="azure_openai",status="success"} 1234
gateway_request_duration_seconds_bucket{provider="azure_openai",le="1.0"} 567

# Provider metrics
gateway_provider_health{provider="azure_openai"} 1  # 1=healthy, 0=unhealthy
gateway_provider_latency_ms{provider="azure_openai"} 245
gateway_provider_error_rate{provider="azure_openai"} 0.02

# Rate limiting metrics
gateway_rate_limit_exceeded_total{user_id="user123"} 5
gateway_rate_limit_remaining{user_id="user123"} 950

# Token metrics
gateway_tokens_used_total{model="gpt-4",provider="azure_openai"} 50000
gateway_cost_total_usd{model="gpt-4"} 1.25
```

### Structured Logging

Logs are output in JSON format with correlation IDs:

```json
{
  "timestamp": "2024-01-15T10:30:45.123Z",
  "level": "INFO",
  "correlation_id": "req-12345-abcde",
  "message": "Request routed to provider",
  "provider": "azure_openai",
  "latency_ms": 245,
  "tokens_used": 125,
  "cost_usd": 0.0075
}
```

### Alerting Examples

```yaml
# Prometheus alert rules
- alert: HighErrorRate
  expr: rate(gateway_requests_total{status="error"}[5m]) > 0.05
  for: 5m
  annotations:
    summary: "High error rate detected"

- alert: ProviderDown
  expr: gateway_provider_health == 0
  for: 2m
  annotations:
    summary: "Provider {{ $labels.provider }} is down"
```

---

## 🧪 Testing

### Run All Tests

```bash
# Run all tests
pytest

# With coverage
pytest --cov=src/gateway --cov-report=html

# Specific test file
pytest tests/unit/test_routing.py -v

# Integration tests only
pytest tests/integration/ -v
```

### Load Testing

```bash
# Using Apache Bench
ab -n 1000 -c 100 http://localhost:8000/health

# Using custom load test
bash scripts/load_test.sh
```

---

## 🐛 Troubleshooting

### Common Issues

**1. Connection refused to Redis**
```bash
# Check Redis is running
redis-cli ping
# Response: PONG
```

**2. Azure OpenAI authentication fails**
```
Verify AZURE_OPENAI_API_KEY and AZURE_OPENAI_ENDPOINT in .env
```

**3. Provider timeout**
```
Increase timeout in config.py: REQUEST_TIMEOUT = 60
```

See [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) for more details.

---

## 📖 Documentation

- [API Documentation](docs/API.md) - Complete API reference
- [Architecture](docs/ARCHITECTURE.md) - Detailed architecture decisions
- [Provider Integration Guide](docs/PROVIDERS.md) - Adding new providers
- [Deployment Guide](docs/DEPLOYMENT.md) - Production deployment
- [Monitoring Guide](docs/MONITORING.md) - Observability setup

---

## 🤝 Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](.github/CONTRIBUTING.md) for guidelines.

### Development Workflow

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Commit changes: `git commit -m 'Add amazing feature'`
4. Push to branch: `git push origin feature/amazing-feature`
5. Open a Pull Request

### Code Standards

- Follow PEP 8 (enforced by Black and Ruff)
- Add type hints to all functions
- Write tests for new features (90%+ coverage)
- Update documentation

```bash
# Format code
black src/ tests/

# Lint
ruff check src/ tests/

# Type check
mypy src/

# Run tests
pytest
```

---

## 📜 License

This project is licensed under the MIT License - see [LICENSE](LICENSE) file for details.

---

## 🙋 Support

- **Issues**: [GitHub Issues](https://github.com/MohitVerma007/genai-api-gateway/issues)
- **Discussions**: [GitHub Discussions](https://github.com/MohitVerma007/genai-api-gateway/discussions)
- **Email**: support@example.com

---

## 🎓 Learning Resources

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Pydantic Documentation](https://docs.pydantic.dev/)
- [Azure OpenAI Docs](https://learn.microsoft.com/en-us/azure/ai-services/openai/)
- [HuggingFace Documentation](https://huggingface.co/docs)
- [Async Python Guide](https://docs.python.org/3/library/asyncio.html)

---

## 🙌 Acknowledgments

- FastAPI community for excellent async framework
- Azure OpenAI for enterprise LLM capabilities
- HuggingFace for open-source model access
- Contributors and users for feedback and improvements

---

<div align="center">
  <strong>Built with ❤️ for the AI community</strong>
  
  [⬆ Back to Top](#-multi-provider-genai-integration-api-gateway)
</div>

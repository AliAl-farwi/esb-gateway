# esb-gateway
Modular, config-driven Enterprise Service Bus with multi-format parsing (JSON/XML/YAML/CSV/GraphQL), webhook &amp; WebSocket engines, and built-in observability (Prometheus, tracing).
# ESB Gateway

**A modular Enterprise Service Bus (ESB) for real-time, multi-format system integration.**

[![Python](https://img.shields.io/badge/Python-3.x-blue.svg)](https://www.python.org/)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED.svg)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## Overview

ESB Gateway is a lightweight, highly modular Enterprise Service Bus built to connect internal and external systems without hard-coding integration logic. Instead of writing custom glue code for every new system, routing, transformation, and protocol handling are all driven by configuration — new integrations are added by writing config, not code.

It was built to solve a recurring problem in enterprise environments: every new system integration (a partner API, an internal service, a legacy data feed) tends to become a one-off script. ESB Gateway centralizes that logic into a single, observable, extensible gateway.

## Key Features

- **Multi-format support** — Dedicated parsers for JSON, XML, YAML, CSV, URL-encoded, and plain text, selected dynamically through a parser factory.
- **Config-driven routing** — Sources, destinations, and transformation rules are declared in configuration; a central router dispatches messages to the right handler.
- **GraphQL endpoints** — Expose unified GraphQL APIs over heterogeneous backend systems.
- **Real-time event handling** — Webhook engine for inbound/outbound event-driven integrations.
- **Persistent connections** — WebSocket engine for systems requiring continuous, low-latency communication.
- **Middleware pipeline** — Pluggable middleware for authentication, rate limiting, request logging, and input validation, applied per route.
- **Connection pooling** — Reused HTTP connections for outbound requests, reducing latency to downstream systems.
- **Concurrent processing** — Dynamic routing engine built on multi-processing, threading, and queuing for high-throughput workloads.
- **Observability** — Prometheus metrics, health-check endpoints, and distributed tracing, alongside structured logging and custom exception handling.
- **Caching** — Built-in caching layer for frequently accessed data and configuration.
- **Test coverage** — Unit tests for parsers and routing, plus integration tests, run via `pytest`.
- **Containerized deployment** — Docker and Docker Compose setup for local and production deployment.

## Request Flow

```
  Inbound request
        │
        ▼
  ┌───────────────────┐    ┌──────────────────────────┐
  │  HTTP Server        │───▶│  Middleware pipeline      │
  │ (server.py,         │    │  auth ▸ rate limit ▸      │
  │  request_handler)   │    │  logging ▸ validation     │
  └───────────────────┘    └────────────┬─────────────┘
                                         ▼
                              ┌────────────────────────┐
                              │   Parser Factory          │
                              │  JSON / XML / YAML /      │
                              │  CSV / URL-encoded / Text │
                              └────────────┬────────────┘
                                           ▼
                              ┌────────────────────────┐
                              │   Router → Handlers        │
                              └────────────┬────────────┘
                                           ▼
                              ┌────────────────────────┐
                              │  Destination Manager       │
                              │  + Forwarder (pooled       │
                              │  connections)               │
                              └────────────┬────────────┘
                                           ▼
                                Destination system(s)

  Cross-cutting: monitoring/ (Prometheus metrics, health checks,
  tracing) and config/ (settings + logging) wrap every request.
```

## Project Structure

```
esp_gateway/
├── main.py                      # Application entry point
├── config/                      # Configuration and logging setup
├── core/                        # Data models, exceptions, constants
├── parsers/                     # Format parsers (JSON, XML, YAML, CSV, URL-encoded, text) + factory
├── routing/
│   ├── router.py                # Main router
│   ├── middleware/               # Auth, rate limiting, logging, validation
│   └── handlers.py              # Route handlers
├── http/                        # HTTP server, connection pool, request handling
├── destinations/                # Destination manager + forwarder
├── monitoring/                  # Prometheus metrics, health checks, tracing
├── utils/                       # Caching, validators, helpers
├── tests/                       # Unit + integration tests (pytest)
├── docker/                      # Dockerfile, docker-compose.yml
├── requirements.txt
├── setup.py
└── .env.example
```

## Tech Stack

| Layer | Technology |
|---|---|
| Core language | Python |
| APIs | REST, GraphQL |
| Messaging | Message Queues, Webhooks, WebSockets |
| Concurrency | Multi-processing, Threading, Queuing, Connection pooling |
| Middleware | Custom auth, rate limiting, logging, and validation middleware |
| Monitoring | Prometheus metrics, health checks, distributed tracing |
| Testing | pytest (unit + integration) |
| Deployment | Docker, Docker Compose, Linux |
| Data formats | JSON, XML, YAML, CSV, URL-encoded, Text |

## Getting Started

### Prerequisites

- Python 3.x
- Docker (recommended for deployment)

### Installation

```bash
git clone https://github.com/AliAl-farwi/esp_gateway.git
cd esp_gateway
pip install -r requirements.txt
```

### Configuration

```bash
cp .env.example .env
```

Application and logging settings live in `config/settings.py` and `config/logging_config.py`. Route definitions are picked up by `routing/router.py` and dispatched to the handlers in `routing/handlers.py`.

### Running locally

```bash
python main.py
```

### Running tests

```bash
pytest tests/
```

### Running with Docker

```bash
docker build -f docker/Dockerfile -t esp-gateway .
docker run -p 8080:8080 esp-gateway
```

Or with Docker Compose:

```bash
docker compose -f docker/docker-compose.yml up
```

## Monitoring & Health

The gateway exposes operational endpoints via `monitoring/`:

- **Health check** — reports service liveness/readiness for orchestration platforms (Docker, Kubernetes, etc.).
- **Metrics** — Prometheus-compatible metrics for request throughput, error rates, and queue depth.
- **Tracing** — distributed tracing hooks for following a message across parsers, middleware, and destinations.

## Project Status

Actively maintained. Current focus: a management and monitoring GUI for visualizing active routes, throughput, and error rates in real time.

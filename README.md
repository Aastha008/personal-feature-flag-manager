# Personal Feature Flag Manager

[![Python Version](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/downloads/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100%2B-009688.svg)](https://fastapi.tiangolo.com/)
[![React](https://img.shields.io/badge/React-18-61DAFB.svg)](https://react.dev/)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)]()
[![Code Style](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A lightweight, self-hosted feature toggle service and interactive management system. Designed for low latency, zero external data leakage, and straightforward deployment, this application enables software engineering teams to control feature rollouts, target specific user segments, and stream configuration changes in real time.

---

## 📋 Table of Contents

- [Problem Statement & Target Audience](#-problem-statement--target-audience)
- [Key Features](#-key-features)
- [Architecture & System Design](#-architecture--system-design)
- [Technology Stack](#-technology-stack)
- [Installation & Virtual Environment Setup](#-installation--virtual-environment-setup)
- [Environment Variables Configuration](#-environment-variables-configuration)
- [Usage: Running the Application & API Reference](#-usage-running-the-application--api-reference)
- [Automated Testing Instructions](#-automated-testing-instructions)
- [Project Structure](#-project-structure)
- [Roadmap & Future Enhancements](#-roadmap--future-enhancements)
- [License](#-license)

---

## 🎯 Problem Statement & Target Audience

### The Problem
Modern continuous delivery requires decoupling software deployments from feature releases. While third-party Feature Flagging SaaS platforms offer robust feature sets, they introduce significant challenges:
- **High Recurring Costs**: Tiered pricing scales aggressively with monthly active users (MAUs) or evaluation requests.
- **Latency Overheads**: External API calls or network hops during flag evaluation can slow down critical request flows.
- **Data Privacy Concerns**: Sending user attributes and telemetry to external third-party cloud vendors risks regulatory and compliance violations (GDPR, HIPAA).
- **Operational Complexity**: Existing open-source alternatives often require enterprise-grade infrastructure (Kubernetes, Kafka, Redis clusters) that is overkill for medium-sized applications.

### Target Audience
- **Engineering Teams**: Looking to perform canary releases, A/B testing, and instant kill-switches without enterprise SaaS bloat.
- **Full-Stack Developers**: Who need a zero-dependency, self-hosted toggle service integrated directly into FastAPI and React stacks.
- **Privacy-Conscious Organizations**: Mandating self-contained data boundaries with SQLite/embedded storage.

---

## ✨ Key Features

- **Interactive Management Dashboard**: Single-page application built with React, TypeScript, and TailwindCSS to create, toggle, and organize feature flags across multiple environments (Development, Staging, Production).
- **Dynamic Flag Evaluation Engine**: High-performance, deterministic evaluation engine supporting MD5 hash-based percentage rollouts, custom user attribute rules (e.g., email domain, user ID, role), and configurable fallback states.
- **Real-Time Streaming Updates (SSE)**: Built-in Server-Sent Events (SSE) broadcaster that pushes real-time flag configuration changes directly to connected client applications, eliminating polling latency.
- **Comprehensive Audit Logging System**: Detailed audit trail tracking every configuration change, toggle history event, and evaluation state modification with administrative context.

---

## 🏗️ Architecture & System Design

The application follows a clean modular layout separating data storage, flag evaluation logic, HTTP API handling, and static front-end assets.

```
                   +---------------------------------------+
                   |       React + Tailwind Dashboard      |
                   +---------------------------------------+
                                       |
                                HTTP / SSE Stream
                                       v
                   +---------------------------------------+
                   |             FastAPI Router            |
                   |             (src/api.py)              |
                   +---------------------------------------+
                     /                 |                 \
                    v                  v                  v
         +--------------------+  +-----------+  +--------------------+
         | Evaluation Engine  |  | Database  |  |  SSE Broadcaster   |
         |    (src/core.py)   |  | (SQLite)  |  |    (src/core.py)   |
         +--------------------+  +-----------+  +--------------------+
                   |                   ^
                   |                   |
                   v                   |
         +-------------------------------------------------------+
         |     Pydantic Schemas & SQLAlchemy ORM Entities        |
         |                    (src/models.py)                    |
         +-------------------------------------------------------+
```

### Module Responsibilities

- **`src/models.py`**: Defines Pydantic domain schemas for API payloads, database models using SQLAlchemy ORM for `Flag`, `Rule`, and `AuditLog` tables.
- **`src/core.py`**: Encapsulates the core business logic: deterministic hash-based percentage calculation, rule attribute matching, and the SSE connection manager (`ConnectionManager`).
- **`src/api.py`**: Contains HTTP routes for administrative CRUD actions, client evaluation endpoints, audit log queries, and the SSE broadcast endpoint.
- **`src/main.py`**: Handles application startup lifecycle events, database initializations, static file mounting for the dashboard, and Uvicorn web server execution.

---

## 🛠️ Technology Stack

| Domain | Technology | Description |
| :--- | :--- | :--- |
| **Backend Framework** | [FastAPI](https://fastapi.tiangolo.com/) | High-performance Python async web framework |
| **ASGI Server** | [Uvicorn](https://www.uvicorn.org/) | Lightning-fast ASGI server implementation |
| **Data Validation** | [Pydantic](https://docs.pydantic.dev/) | Strict typing and data serialization |
| **Database & ORM** | SQLite + [SQLAlchemy](https://www.sqlalchemy.org/) | Embedded relational database with ORM abstraction |
| **Frontend UI** | React + TypeScript + TailwindCSS | Interactive SPA administrative dashboard |
| **Testing** | [pytest](https://docs.pytest.org/) | Suite for automated unit and integration tests |

---

## ⚙️ Installation & Virtual Environment Setup

### Prerequisites
- Python 3.10 or higher
- `pip` (Python package installer)

### Step-by-Step Setup

1. **Clone the Repository**
   ```bash
   git clone https://github.com/your-org/personal-feature-flag-manager.git
   cd personal-feature-flag-manager
   ```

2. **Create a Virtual Environment**
   ```bash
   python -m venv venv
   ```

3. **Activate the Virtual Environment**
   - **Linux / macOS**:
     ```bash
     source venv/bin/activate
     ```
   - **Windows (PowerShell)**:
     ```powershell
     .\venv\Scripts\Activate.ps1
     ```

4. **Install Dependencies**
   ```bash
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

---

## 🔑 Environment Variables Configuration

Create a `.env` file in the root directory by copying the example environment file:

```bash
cp .env.example .env
```

### Available Environment Variables

| Variable | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `DATABASE_URL` | String | `sqlite:///./feature_flags.db` | Connection string for SQLite database |
| `API_PREFIX` | String | `/api/v1` | URL prefix for REST endpoints |
| `DEBUG` | Boolean | `True` | Enable verbose debugging and auto-reload |
| `HOST` | String | `127.0.0.1` | Network interface binding host |
| `PORT` | Integer | `8000` | Network port for Uvicorn server |
| `SECRET_KEY` | String | `change-me-in-production` | Secret key for signing/verification operations |

---

## 🚀 Usage: Running the Application & API Reference

### Starting the Server

Run the application using Python's module launcher:

```bash
python -m src.main
```

Alternatively, launch directly via Uvicorn:

```bash
uvicorn src.main:app --host 127.0.0.1 --port 8000 --reload
```

Once running, access:
- **Admin Dashboard**: `http://127.0.0.1:8000/`
- **Interactive OpenAPI Specs**: `http://127.0.0.1:8000/docs`

---

### REST API Endpoints Reference

#### 1. Create a Feature Flag
- **Endpoint**: `POST /api/v1/flags`
- **Payload**:
  ```json
  {
    "key": "new_checkout_flow",
    "name": "New Checkout Flow",
    "description": "Enables the redesigned multi-step checkout UI",
    "enabled": true,
    "environment": "production",
    "percentage_rollout": 50,
    "rules": [
      {
        "attribute": "email",
        "operator": "ends_with",
        "value": "@company.com"
      }
    ]
  }
  ```

#### 2. Evaluate Flag for Context
- **Endpoint**: `POST /api/v1/evaluate`
- **Payload**:
  ```json
  {
    "flag_key": "new_checkout_flow",
    "environment": "production",
    "context": {
      "user_id": "usr_99812",
      "email": "developer@company.com"
    }
  }
  ```
- **Response**:
  ```json
  {
    "flag_key": "new_checkout_flow",
    "value": true,
    "reason": "MATCHED_RULE",
    "eval_time_ms": 0.42
  }
  ```

#### 3. Real-Time SSE Stream Endpoint
- **Endpoint**: `GET /api/v1/stream`
- **Description**: Establishes a persistent Server-Sent Events channel. Broadcasts event objects when flag configurations update.

#### 4. Fetch Audit Logs
- **Endpoint**: `GET /api/v1/audit-logs?limit=20`
- **Response**:
  ```json
  [
    {
      "id": 1,
      "flag_key": "new_checkout_flow",
      "action": "TOGGLE_ON",
      "timestamp": "2023-10-25T14:32:00Z",
      "details": "Flag enabled by Admin"
    }
  ]
  ```

---

## 🧪 Automated Testing Instructions

The repository includes test suites covering evaluation logic, hashing determinism, dynamic targeting rules, and FastAPI API routes.

### Run All Tests

```bash
pytest
```

### Run Tests with Verbose Output

```bash
pytest -v
```

### Run Core Engine Tests Only

```bash
pytest tests/test_core.py
```

### Run API Route Tests Only

```bash
pytest tests/test_api.py
```

### Check Test Coverage

```bash
pytest --cov=src --cov-report=term-missing
```

---

## 📁 Project Structure

```
personal-feature-flag-manager/
├── .env.example            # Template for environment configuration
├── .gitignore              # Git ignore rules
├── requirements.txt        # Python package dependencies
├── src/
│   ├── __init__.py         # Package initialization
│   ├── api.py              # FastAPI routers for admin CRUD, evaluation & SSE
│   ├── core.py             # Evaluation engine & SSE connection management
│   ├── main.py             # Server lifecycle, app creation & static mounting
│   └── models.py           # Pydantic schemas & SQLAlchemy ORM entities
└── tests/
    ├── __init__.py         # Test package initialization
    ├── test_api.py         # HTTP endpoint tests
    └── test_core.py        # Evaluation engine and calculation unit tests
```

---

## 🛠️ Roadmap & Future Enhancements

- [ ] **Redis Caching Layer**: Add optional in-memory caching layer to eliminate SQLite reads for ultra-high-throughput setups.
- [ ] **Multi-Tenant RBAC**: Implement role-based access control (Admin, Developer, Viewer) with API key authentication for multiple teams.
- [ ] **Client SDKs**: Publish lightweight SDK wrappers for Python, Node.js/TypeScript, and React (Hooks).
- [ ] **Webhook Notifications**: Push configuration change notifications to Slack, Teams, or custom HTTP webhooks.

---

## 📄 License

Distributed under the MIT License. See `LICENSE` for more information.
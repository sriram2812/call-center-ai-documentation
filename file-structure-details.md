# Microsoft Call Center AI - File Structure Details

This document provides an in-depth explanation of the file structure of the Microsoft Call Center AI project, detailing the purpose and functionality of each key file.

## Repository Root

### Core Files

| File | Description |
|------|-------------|
| `README.md` | Project overview, features, deployment instructions, and architecture diagrams |
| `Makefile` | Automation commands for building, testing, and deploying the application |
| `pyproject.toml` | Python project configuration, including dependencies and build settings |
| `config-local-example.yaml` | Example configuration for local development environment |
| `config-remote-example.yaml` | Example configuration for Azure cloud deployment |
| `.env.example` | Example environment variables for local development |

## App Directory (`/app`)

The core application code is organized into several directories based on functionality.

### Main Application File

| File | Description |
|------|-------------|
| `app/main.py` | The entry point of the application. It sets up the FastAPI server, defines API endpoints, handles WebSocket connections for real-time audio streaming, and manages communication with Azure services. |

### Models Directory (`/app/models`)

Data structures and schema definitions used throughout the application.

| File | Description |
|------|-------------|
| `app/models/call.py` | Defines the data models for calls, including `CallInitiateModel` (parameters for starting a call), `CallGetModel` (public data for API responses), and `CallStateModel` (complete call state with conversation history) |
| `app/models/claim.py` | Schema for structured data collected during calls, supports various field types like text, datetime, email, and phone numbers |
| `app/models/message.py` | Represents conversation messages, including content, speaker information, and message type |
| `app/models/next.py` | Defines possible next actions after a call completes (e.g., callback, transfer to agent) |
| `app/models/readiness.py` | Health check models for service monitoring |
| `app/models/reminder.py` | Follow-up task data structure for tasks generated during calls |
| `app/models/synthesis.py` | Call summary and insights model generated at the end of conversations |
| `app/models/training.py` | Structures for training data used in the RAG system |

### Helpers Directory (`/app/helpers`)

Utility functions and service abstractions to support the core functionality.

| File | Description |
|------|-------------|
| `app/helpers/cache.py` | Caching utilities to improve performance |
| `app/helpers/call_events.py` | Handles call lifecycle events (connection, disconnection, etc.) |
| `app/helpers/call_llm.py` | Interface to language models for generating responses |
| `app/helpers/call_utils.py` | Common utilities for call processing |
| `app/helpers/config.py` | Configuration management using a nested object structure |
| `app/helpers/features.py` | Feature flag management for controlling behavior |
| `app/helpers/http.py` | HTTP client utilities with connection pooling |
| `app/helpers/identity.py` | Authentication and identity management |
| `app/helpers/llm_tools.py` | Function calling tools available to the language model |
| `app/helpers/llm_utils.py` | Utilities for working with language models |
| `app/helpers/llm_worker.py` | Background processing for language model tasks |
| `app/helpers/logging.py` | Logging configuration and utilities |
| `app/helpers/monitoring.py` | Telemetry and application insights integration |
| `app/helpers/resources.py` | Resource management for static files |
| `app/helpers/translation.py` | Language translation services |

#### Config Models Directory (`/app/helpers/config_models`)

| File | Description |
|------|-------------|
| `app/helpers/config_models/conversation.py` | Configuration models for conversation settings |
| `app/helpers/config_models/llm.py` | Configuration for language model integration |
| `app/helpers/config_models/persistence.py` | Configuration for storage services |

#### Pydantic Types Directory (`/app/helpers/pydantic_types`)

| File | Description |
|------|-------------|
| `app/helpers/pydantic_types/phone_numbers.py` | Custom phone number validation and formatting |

### Persistence Directory (`/app/persistence`)

Interfaces to external storage and services for data persistence.

| File | Description |
|------|-------------|
| `app/persistence/ai_search.py` | Azure AI Search integration for RAG capabilities |
| `app/persistence/azure_queue_storage.py` | Queue management for asynchronous processing |
| `app/persistence/communication_services.py` | Azure Communication Services integration for calls and SMS |
| `app/persistence/cosmos_db.py` | Database operations for storing calls and claims |
| `app/persistence/icache.py` | Interface definition for cache implementations |
| `app/persistence/isearch.py` | Interface definition for search implementations |
| `app/persistence/isms.py` | Interface definition for SMS implementations |
| `app/persistence/istore.py` | Interface definition for data storage implementations |
| `app/persistence/memory.py` | In-memory storage for development and testing |
| `app/persistence/redis.py` | Redis integration for distributed caching |
| `app/persistence/twilio.py` | Twilio integration for alternative SMS capabilities |

### Resources Directory (`/app/resources`)

Static resources and assets used by the application.

| File | Description |
|------|-------------|
| `app/resources/public_website` | Web templates for the user interface |
| `app/resources/tiktoken` | Tokenization resources for the language model |

## CI/CD Directory (`/cicd`)

Infrastructure as Code (IaC) and deployment resources.

| File | Description |
|------|-------------|
| `cicd/Dockerfile` | Container definition for the application |
| `cicd/bicep/app.bicep` | Azure resource definitions for the application |
| `cicd/bicep/main.bicep` | Main Bicep file orchestrating deployment |

## Tests Directory (`/tests`)

Test cases and testing utilities.

| File | Description |
|------|-------------|
| `tests/local.py` | Local testing script without requiring Communication Services |
| `tests/unit` | Unit tests for components |
| `tests/integration` | Integration tests for services |

## Doc Directory (`/docs`)

Documentation and additional resources.

| File | Description |
|------|-------------|
| `docs/user_report.png` | Screenshot of the user report interface |

## Development Configuration (`.devcontainer`, `.vscode`)

Settings for development environments.

| File | Description |
|------|-------------|
| `.devcontainer` | GitHub Codespaces configuration for instant development environment |
| `.vscode` | VS Code settings and recommended extensions |

## Code Flow and Interactions

### Call Flow

1. An incoming call is received via Azure Communication Services
2. The event is captured in `app/main.py` and routed to `call_event` function
3. `call_events.py` processes the call with `on_new_call`
4. A WebSocket connection is established for real-time audio streaming
5. Audio from the caller is processed by Speech-to-Text
6. The transcript is sent to the language model via `call_llm.py`
7. The model response is converted to speech and sent back to the caller
8. The conversation is stored in Cosmos DB for future reference
9. At the end of the call, `on_end_call` generates insights and next actions

### REST API Endpoints

The main API endpoints defined in `app/main.py`:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/call` | POST | Initiate an outbound call |
| `/call` | GET | List all calls |
| `/call/{call_id_or_phone_number}` | GET | Get a specific call by ID or phone number |
| `/health/liveness` | GET | Service health check (liveness probe) |
| `/health/readiness` | GET | Service readiness check (readiness probe) |
| `/report` | GET | Web interface to list calls |
| `/report/{call_id}` | GET | Web interface to view a specific call |
| `/communicationservices/callback/{call_id}/{secret}` | POST | Callback for Communication Services events |
| `/communicationservices/wss/{call_id}/{secret}` | WebSocket | WebSocket endpoint for real-time audio streaming |

### Asynchronous Processing

The application uses several queues for asynchronous processing:

1. **Call Queue**: Processes incoming call events
2. **Post Queue**: Handles post-call processing like generating summaries
3. **SMS Queue**: Processes incoming SMS messages
4. **Training Queue**: Manages RAG data updates

These queues are configured in the application's lifespan handler and processed by dedicated worker functions.
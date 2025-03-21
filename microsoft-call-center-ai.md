# Microsoft Call Center AI - Comprehensive Documentation

This document provides a detailed overview of Microsoft's Call Center AI project, explaining each component, its functionality, and how everything works together to create an AI-powered call center solution.

## Table of Contents

1. [Introduction](#introduction)
2. [System Overview](#system-overview)
3. [Key Features](#key-features)
4. [Architecture](#architecture)
5. [Core Components](#core-components)
6. [File Structure Explained](#file-structure-explained)
7. [Deployment](#deployment)
8. [Customization](#customization)
9. [Use Cases](#use-cases)

## Introduction

Microsoft Call Center AI is an advanced AI-powered call center solution that combines Azure services with OpenAI GPT models to create a system capable of handling automated phone calls. The system can be used for inbound and outbound calls, supporting multiple languages, and can be customized for various scenarios like insurance claims, IT support, customer service, and more.

The solution leverages state-of-the-art technologies like GPT-4o and Azure Cognitive Services to provide a natural conversation experience while collecting structured data during calls. All conversations are streamed in real-time, can be resumed after disconnections, and are stored for future reference.

## System Overview

The system allows you to:

1. Make outbound calls to customers via an API call
2. Receive inbound calls through a configured phone number
3. Conduct natural conversations with customers
4. Collect structured data through a claim schema
5. Generate summaries and to-do items from conversations
6. Store and retrieve conversation history
7. Transfer calls to human agents when needed
8. Send follow-up SMS messages

## Key Features

### Enhanced Communication and User Experience
- Inbound and outbound calls with a dedicated phone number
- Support for multiple languages and voice tones
- Real-time streaming of conversations to avoid delays
- Ability to resume conversations after disconnections
- 24/7 communication capabilities
- Integration with SMS for additional information sharing

### Advanced Intelligence and Data Management
- Utilizes GPT-4o and GPT-4o Realtime for natural language understanding
- Can discuss private and sensitive data including customer-specific information
- Follows Retrieval-Augmented Generation (RAG) best practices
- Understands domain-specific terms
- Follows structured claim schema
- Generates automated to-do lists
- Filters inappropriate content and detects jailbreak attempts
- Historical conversations can be used to fine-tune the LLM

### Customization, Oversight, and Scalability
- Customizable prompts and voice options
- Feature flags for controlled experimentation
- Human agent fallback
- Call recording for quality assurance
- Integration with Application Insights for monitoring
- Publicly accessible claim data
- Brand-specific custom voice support

### Cloud-Native Deployment
- Deployed on Azure with a containerized, serverless architecture
- Low maintenance requirements with elastic scaling
- Optimized costs based on usage
- Seamless integration with Azure services

## Architecture

The system architecture follows a modern cloud-native design with several key components working together:

### High Level Architecture
```
User <-- Call --> Call Center AI <-- Transfer to --> Agent
```

### Component Level Architecture
The system consists of the following main components:
- **App Container**: The core application logic (Container App)
- **Embedding**: Text embedding service (ADA)
- **Call & SMS Gateway**: Communication interface (Communication Services)
- **Database**: Conversations and claims storage (Cosmos DB)
- **Event Broker**: Message coordination (Event Grid)
- **LLM**: Language processing (GPT-4o)
- **Queue**: Message processing (Azure Storage)
- **Cache**: Performance optimization (Redis)
- **RAG**: Retrieval augmentation (AI Search)
- **Sound Storage**: Audio files (Azure Storage)
- **Speech Services**: Audio processing (Cognitive Services)
  - Speech-to-text
  - Text-to-speech
  - Translation

## Core Components

### Communication Services
Handles all phone call and SMS interactions using Azure Communication Services. It enables:
- Inbound and outbound calls
- Real-time audio streaming
- Call transfers to human agents
- SMS capabilities for follow-up

### Language Models
Uses OpenAI's GPT-4o and GPT-4o Realtime models to:
- Process natural language input
- Generate human-like responses
- Follow structured conversation flows
- Extract claim information
- Create summaries and next actions

### Speech Services
Leverages Azure Cognitive Services for:
- Converting speech to text for processing
- Converting text responses back to speech
- Supporting multiple languages
- Custom voice capability

### Retrieval-Augmented Generation
Integrates Azure AI Search and vector embeddings to:
- Enhance responses with relevant information
- Provide domain-specific knowledge
- Access appropriate reference materials

### Data Storage
Uses multiple storage solutions:
- Cosmos DB for conversation and claim data
- Azure Storage for queues and audio files
- Redis for caching and performance optimization

### Monitoring and Analytics
Implements monitoring using:
- Application Insights for telemetry
- Custom metrics for call quality
- Comprehensive logging

## File Structure Explained

The repository is organized into several key directories and files:

### `/app` Directory
The core application code.

#### `/app/main.py`
The entrypoint of the application which:
- Sets up the FastAPI server
- Defines all API endpoints
- Handles request routing
- Manages WebSocket connections
- Processes Communication Services events

#### `/app/models` Directory
Contains data models that define the structure of:

- **`call.py`**: Manages call state and information
  - `CallInitiateModel`: Parameters for initiating a call
  - `CallStateModel`: The complete state of a call including messages and claims
  - `CallGetModel`: The public-facing call data for API responses

- **`claim.py`**: Defines the structure for claim data collected during calls

- **`message.py`**: Represents conversation messages
  - Tracks speaker (human or assistant)
  - Stores message content and metadata
  - Handles different message actions (talk, call, hangup)

- **`next.py`**: Defines possible next actions for a call
  - Used to determine how to handle a call after completion

- **`reminder.py`**: Stores follow-up tasks generated during calls

- **`synthesis.py`**: Contains conversation summaries and insights

- **`training.py`**: Manages training data for the language model

#### `/app/helpers` Directory
Utility functions and service interfaces:

- **`call_events.py`**: Processes events during a call (connect, disconnect, recognition)

- **`call_llm.py`**: Interfaces with the OpenAI GPT models

- **`call_utils.py`**: Common utilities for call processing

- **`features.py`**: Feature flag management

- **`llm_tools.py`**: Tools available to the language model

- **`llm_utils.py`**: Utilities for working with language models

- **`llm_worker.py`**: Background processing for language model tasks

- **`cache.py`**: Caching implementation

- **`config.py`**: Application configuration management

- **`http.py`**: HTTP client utilities

- **`logging.py`**: Logging configuration

- **`monitoring.py`**: Telemetry and monitoring

- **`translation.py`**: Language translation services

#### `/app/persistence` Directory
Interfaces to external storage and services:

- **`ai_search.py`**: Azure AI Search integration for RAG

- **`azure_queue_storage.py`**: Queue management for asynchronous processing

- **`communication_services.py`**: Azure Communication Services interface

- **`cosmos_db.py`**: Database operations for storing calls and claims

- **`redis.py`**: Caching layer for performance optimization

- **`twilio.py`**: Twilio integration for SMS (alternative to Azure)

### `/cicd` Directory
Contains deployment and infrastructure files:

#### `/cicd/bicep`
- **`app.bicep`**: Defines Azure resources for the application
- **`main.bicep`**: Entry point for infrastructure deployment

#### `/cicd/Dockerfile`
Container definition for the application

### Configuration Files
- **`config-local-example.yaml`**: Example configuration for local development
- **`config-remote-example.yaml`**: Example configuration for cloud deployment
- **`pyproject.toml`**: Python project configuration
- **`Makefile`**: Automation scripts for development and deployment

## Deployment

The system can be deployed in two ways:

### Remote Deployment on Azure
1. Create a resource group
2. Create a Communication Services resource
3. Buy a phone number with voice capabilities
4. Create a configuration file based on the example
5. Deploy using the Makefile command: `make deploy name=my-rg-name`

### Local Development
1. Install required tools (Rust, uv)
2. Create a configuration file
3. Deploy Azure resources: `make deploy-bicep deploy-post name=my-rg-name`
4. Set up a dev tunnel: `make tunnel`
5. Run the application: `make dev`

## Customization

The system offers extensive customization options:

### Claim Schema
You can define custom data fields to collect during calls:
- Different field types (text, datetime, email, phone_number)
- Optional descriptions for each field

### Language Support
Configure multiple languages with:
- Default language setting
- Voice selection per language
- Custom Neural Voice integration

### Prompts
Customize the conversation flow with:
- System prompts to define the assistant's behavior
- Greeting templates
- Response variations for natural conversation

### Call Objectives
Define specific goals for each call type, allowing the system to:
- Focus on particular data collection
- Provide specialized assistance
- Follow custom workflows

### Feature Flags
Control behavior through configuration:
- Timeout settings
- Voice activity detection parameters
- Recording options
- Model selection

## Use Cases

Microsoft Call Center AI can be applied to various scenarios:

1. **Insurance Claims Processing**
   - Collect initial claim information
   - Schedule follow-up calls
   - Send confirmation details via SMS

2. **IT Support**
   - Troubleshoot common issues
   - Collect system information
   - Escalate complex cases to human agents

3. **Customer Service**
   - Answer frequently asked questions
   - Process returns or exchanges
   - Update account information

4. **Appointment Scheduling**
   - Book new appointments
   - Confirm or reschedule existing appointments
   - Send reminders

5. **Surveys and Feedback**
   - Collect customer feedback
   - Conduct satisfaction surveys
   - Gather market research data
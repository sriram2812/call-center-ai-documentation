# Microsoft Call Center AI - Architecture Diagrams

This document contains architectural diagrams explaining the structure and flow of the Microsoft Call Center AI solution.

## High Level Architecture

```mermaid
graph TD
    User([User]) <--> |Phone Call| CallCenter[Call Center AI]
    CallCenter --> |Transfer if needed| Agent([Human Agent])
    
    subgraph Legend
        User-Icon([User]) --- UserDesc[End User/Caller]
        AI-Icon[AI System] --- AIDesc[Call Center AI]
        Agent-Icon([Agent]) --- AgentDesc[Human Agent]
    end
```

## Component Level Architecture

```mermaid
graph TB
    User([User])
    Agent([Human Agent])

    subgraph "Call Center AI"
        ContainerApp[App Container App]
        CommunicationServices[Communication Services]
        OpenAI[GPT-4o Model]
        CosmosDB[(Cosmos DB)]
        Redis[(Redis Cache)]
        AISearch[(AI Search)]
        AzureStorage[(Azure Storage)]
        SpeechSTT[Speech-to-Text]
        SpeechTTS[Text-to-Speech]
        EventGrid[Event Grid]
        ADA[Embedding ADA]
    end

    User <--> |Phone Call| CommunicationServices
    Agent <--> |Transfer| CommunicationServices

    CommunicationServices <--> ContainerApp
    ContainerApp --> |Store Conversation| CosmosDB
    ContainerApp --> |Generate Responses| OpenAI
    OpenAI --> |Return Completion| ContainerApp
    ContainerApp --> |Cache| Redis
    ContainerApp --> |Search Knowledge| AISearch
    AISearch --> |Generate Embeddings| ADA
    ContainerApp --> |Audio Processing| SpeechSTT
    ContainerApp --> |Text to Voice| SpeechTTS
    CommunicationServices --> |Events| EventGrid
    EventGrid --> |Message Queue| AzureStorage
    AzureStorage --> |Process Messages| ContainerApp
```

## Call Flow Sequence

```mermaid
sequenceDiagram
    participant User as User/Caller
    participant ACS as Azure Communication Services
    participant App as Container App
    participant STT as Speech-to-Text
    participant LLM as GPT-4o Model
    participant TTS as Text-to-Speech
    participant DB as Cosmos DB
    participant RAG as AI Search
    
    User->>ACS: Initiates call
    ACS->>App: Call event
    App->>DB: Create call record
    
    loop Conversation
        ACS->>App: Stream audio
        App->>STT: Convert speech to text
        STT->>App: Return transcription
        App->>RAG: Search for relevant information
        RAG->>App: Return context
        App->>LLM: Send transcription + context
        LLM->>App: Generate response
        App->>TTS: Convert response to speech
        TTS->>App: Return audio
        App->>ACS: Stream audio response
        ACS->>User: Deliver response
        App->>DB: Update conversation
    end
    
    User->>ACS: End call
    ACS->>App: Call ended event
    App->>LLM: Generate insights
    LLM->>App: Return summary & next actions
    App->>DB: Save final state
```

## Data Flow

```mermaid
graph LR
    subgraph Input
        Audio[Audio Input]
        SMS[SMS Input]
    end
    
    subgraph Processing
        STT[Speech-to-Text]
        NLP[Natural Language Processing]
        RAG[Retrieval Augmented Generation]
        LLM[Language Model]
    end
    
    subgraph Output
        VoiceResponse[Voice Response]
        TextSummary[Text Summary]
        ClaimData[Structured Claim Data]
        ToDoList[Todo List]
    end
    
    subgraph Storage
        ConversationHistory[(Conversation History)]
        ClaimDatabase[(Claim Database)]
        KnowledgeBase[(Knowledge Base)]
    end
    
    Audio --> STT
    SMS --> NLP
    STT --> NLP
    NLP --> RAG
    KnowledgeBase --> RAG
    RAG --> LLM
    ConversationHistory --> LLM
    LLM --> VoiceResponse
    LLM --> TextSummary
    LLM --> ClaimData
    LLM --> ToDoList
    LLM --> ConversationHistory
    ClaimData --> ClaimDatabase
```

## Deployment Architecture

```mermaid
graph TB
    subgraph Azure
        subgraph "Resource Group"
            CA[Container App]
            ACS[Communication Services]
            CS[Cognitive Services]
            AOAI[Azure OpenAI]
            AIS[Azure AI Search]
            CDB[Cosmos DB]
            RDC[Redis Cache]
            AST[Azure Storage]
            EG[Event Grid]
        end
    end
    
    subgraph "Development Environment"
        Dev[Developer]
        GH[GitHub]
        CI[CI/CD Pipeline]
    end
    
    Dev -->|Push Code| GH
    GH -->|Trigger| CI
    CI -->|Deploy| CA
    
    CA <-->|Communicate| ACS
    CA <-->|Process Speech| CS
    CA <-->|Generate Text| AOAI
    CA <-->|RAG Queries| AIS
    CA <-->|Store Data| CDB
    CA <-->|Cache| RDC
    CA <-->|Queue Messages| AST
    ACS -->|Events| EG
    EG -->|Notifications| AST
```
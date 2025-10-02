# Core Services

> **Relevant source files**
> * [src/java/com/comitfs/openfire/TeamsService.java](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TeamsService.java)

This document covers the main service components that form the backbone of the CAS (comitFS application services). These core services handle the primary business logic, external integrations, and API exposure for the entire platform.

For information about the test automation capabilities, see [Test Automation Framework](./3-test-automation-framework.md). For details about real-time communication features, see [Real-time Communication](./4-real-time-communication.md). For client application interfaces, see [Client Applications](./5-client-applications.md).

## Overview

The CAS system is built around three primary core services that work together to provide a unified communication platform:

```mermaid
flowchart TD

TS["TeamsService<br>Primary Integration Hub"]
CCS["CasCompanionService<br>REST API Layer"]
PA["Plugin Architecture<br>Configuration & Deployment"]
MSGraph["Microsoft Graph API"]
ACS["Azure Communication Services"]
MSTeams["Microsoft Teams"]
Intelliflo["Intelliflo CRM"]
WhatsApp["WhatsApp Business API"]
OF["Openfire XMPP Server"]
AdminConsole["Admin Console Plugin"]
UserManager["User Manager"]
AuthCheck["Auth Check Filter"]

TS --> MSGraph
TS --> ACS
TS --> MSTeams
TS --> Intelliflo
TS --> WhatsApp
PA --> OF
PA --> AdminConsole
PA --> UserManager
PA --> AuthCheck
OF --> TS
OF --> CCS

subgraph subGraph2 ["Openfire Platform"]
    OF
    AdminConsole
    UserManager
    AuthCheck
end

subgraph subGraph1 ["External Integrations"]
    MSGraph
    ACS
    MSTeams
    Intelliflo
    WhatsApp
end

subgraph subGraph0 ["Core Services Architecture"]
    TS
    CCS
    PA
    CCS --> TS
end
```

**Core Service Responsibilities**

| Service | Purpose | Key Integrations |
| --- | --- | --- |
| `TeamsService` | Primary integration hub handling Microsoft services, call automation, and user management | Graph API, ACS, Teams, Bot Framework |
| `CasCompanionService` | REST API layer exposing system functionality to external clients | Jersey framework, HTTP servlets |
| Plugin Architecture | Openfire plugin structure providing configuration, deployment, and lifecycle management | Openfire server, web contexts, authentication |

Sources: [src/java/com/comitfs/openfire/TeamsService.java L188-L400](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TeamsService.java#L188-L400)

## TeamsService Integration Hub

The `TeamsService` class serves as the central coordination point for all external integrations and core business logic. It implements the `ProcessListener` interface and manages multiple client connections, authentication flows, and event processing.

```mermaid
flowchart TD

EventProcessor["eventProcessorClient<br>EventProcessorClient"]
TS["TeamsService"]
GraphClient["graphClient<br>GraphServiceClient"]
ArchiveClient["archiveClient<br>GraphServiceClient"]
EmailClient["emailClient<br>EmailClient"]
CallClient["callAutomationClient<br>CallAutomationClient"]
SmsClient["smsClient<br>SmsClient"]
IdentityClient["communicationIdentityClient<br>CommunicationIdentityClient"]
TokensLookup["tokensLookup<br>Map"]
PhonesLookup["phonesLookup<br>Map"]
ContactSessions["contactSessions<br>Map"]
CallIds["callIds<br>HashMap"]
RelayListener["relayListener<br>HybridConnectionListener"]
TestPlanner["testPlanner<br>TestPlanner"]

subgraph subGraph3 ["TeamsService Core Components"]
    TS
    TS --> GraphClient
    TS --> ArchiveClient
    TS --> EmailClient
    TS --> CallClient
    TS --> SmsClient
    TS --> IdentityClient
    TS --> TokensLookup
    TS --> PhonesLookup
    TS --> ContactSessions
    TS --> CallIds
    TS --> EventProcessor
    TS --> RelayListener
    TS --> TestPlanner

subgraph subGraph2 ["Event Processing"]
    EventProcessor
    RelayListener
    TestPlanner
end

subgraph subGraph1 ["Data Management"]
    TokensLookup
    PhonesLookup
    ContactSessions
    CallIds
end

subgraph subGraph0 ["Authentication & Clients"]
    GraphClient
    ArchiveClient
    EmailClient
    CallClient
    SmsClient
    IdentityClient
end
end
```

**Key Service Methods**

| Method Category | Key Methods | Purpose |
| --- | --- | --- |
| Initialization | `initializeService()`, `destroyService()` | Plugin lifecycle management |
| Graph API | `setupGraphAPI()`, `createGraphAPI()`, `requestSubscriptions()` | Microsoft Graph integration |
| Call Management | `updateCall()`, `makeCall()`, `processUserAction()` | Call status and control |
| User Management | `setupUserProfiles()`, `getUserProfile()`, `ensureUserExists()` | User synchronization |
| ACS Integration | `setupACS()`, `makeOmniCall()`, `establishStreaming()` | Azure Communication Services |

Sources: [src/java/com/comitfs/openfire/TeamsService.java L305-L381](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TeamsService.java#L305-L381)

 [src/java/com/comitfs/openfire/TeamsService.java L408-L473](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TeamsService.java#L408-L473)

 [src/java/com/comitfs/openfire/TeamsService.java L3114-L3271](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TeamsService.java#L3114-L3271)

### Authentication and Token Management

The service implements sophisticated authentication flows for Microsoft services using MSAL (Microsoft Authentication Library) and device code flows:

```mermaid
flowchart TD

User["User Authentication Request"]
DeviceCode["Device Code Flow<br>acquireTokenDeviceCode()"]
TokenCache["Token Cache Aspect<br>TokenCacheAspect"]
SilentAuth["Silent Authentication<br>SilentParameters"]
InteractiveAuth["Interactive Authentication<br>DeviceCodeFlowParameters"]
GraphClient["Graph Service Client<br>GraphServiceClient"]

User --> DeviceCode
DeviceCode --> TokenCache
TokenCache --> SilentAuth
SilentAuth --> GraphClient
SilentAuth --> InteractiveAuth
InteractiveAuth --> GraphClient
```

The authentication process stores tokens in the `tokensLookup` map and sends device code prompts to users via email and real-time notifications.

Sources: [src/java/com/comitfs/openfire/TeamsService.java L858-L922](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TeamsService.java#L858-L922)

 [src/java/com/comitfs/openfire/TeamsService.java L248-L251](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TeamsService.java#L248-L251)

### Call Status Management

TeamsService maintains call state through the Openlink protocol, tracking calls across multiple participants and states:

```mermaid
stateDiagram-v2
    [*] --> CallOriginated
    [*] --> CallDelivered
    CallOriginated --> CallEstablished
    CallOriginated --> CallFailed
    CallOriginated --> ConnectionCleared
    CallDelivered --> CallEstablished
    CallDelivered --> CallMissed
    CallDelivered --> CallFailed
    CallEstablished --> CallHeld
    CallEstablished --> CallConferenced
    CallEstablished --> ConnectionCleared
    CallHeld --> CallEstablished
    CallConferenced --> CallEstablished
    CallConferenced --> ConnectionCleared
    CallMissed --> [*]
    CallFailed --> [*]
    ConnectionCleared --> [*]
```

Sources: [src/java/com/comitfs/openfire/TeamsService.java L4036-L4095](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TeamsService.java#L4036-L4095)

 [src/java/com/comitfs/openfire/TeamsService.java L3426-L3509](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TeamsService.java#L3426-L3509)

## REST API Layer

The REST API layer, implemented through the `CasCompanionService`, provides HTTP endpoints for external clients to interact with the CAS system. This service is built on the Jersey framework and integrates with Openfire's plugin architecture.

```mermaid
flowchart TD

Client["External Clients"]
BasicAuth["Basic Authentication<br>BasicAuth"]
JerseyWrapper["Jersey REST Framework<br>JerseyWrapper.SERVLET_URL"]
CCS["CasCompanionService<br>REST Endpoints"]
UserEnsure["ensureUserExists()<br>User Management"]
BusinessLogic["Business Logic<br>Delegation to TeamsService"]
TS["TeamsService<br>Core Processing"]
UserManager["UserManager<br>Openfire Users"]
EventSource["WebEventSourceServlet<br>Real-time Events"]

JerseyWrapper --> CCS
BusinessLogic --> TS
UserEnsure --> UserManager

subgraph subGraph2 ["Backend Services"]
    TS
    UserManager
    EventSource
    TS --> EventSource
end

subgraph subGraph1 ["REST Service Layer"]
    CCS
    UserEnsure
    BusinessLogic
    CCS --> UserEnsure
    CCS --> BusinessLogic
end

subgraph subGraph0 ["HTTP Layer"]
    Client
    BasicAuth
    JerseyWrapper
    Client --> BasicAuth
    BasicAuth --> JerseyWrapper
end
```

**Authentication and Authorization**

The REST API uses basic authentication and integrates with Openfire's user management system. The service automatically creates users if they don't exist and validates credentials against the Openfire user store.

**Key API Endpoints**

| Endpoint Pattern | Purpose | Authentication |
| --- | --- | --- |
| `/casapi/v1/*` | Main REST API endpoints | Basic Auth |
| `/casapi/sse` | Server-Sent Events | Excluded from auth |
| `/casapi/notify-call-records` | Webhook callbacks | Excluded from auth |
| `/casapi/docs/*` | API documentation | Excluded from auth |

Sources: [src/java/com/comitfs/openfire/TeamsService.java L371-L380](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TeamsService.java#L371-L380)

 [src/java/com/comitfs/openfire/TeamsService.java L296-L302](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TeamsService.java#L296-L302)

## Plugin Architecture & Configuration

The CAS system is implemented as an Openfire plugin, providing deep integration with the XMPP server infrastructure while maintaining modularity and configurability.

```mermaid
flowchart TD

PluginManager["PluginManager<br>Plugin Lifecycle"]
AdminConsole["AdminConsolePlugin<br>Web Interface"]
AuthFilter["AuthCheckFilter<br>Security"]
TeamsService["TeamsService<br>Core Business Logic"]
WebContext["WebAppContext<br>/casweb"]
WSContext["ServletContextHandler<br>/stream-ws"]
TestPlanner["TestPlanner<br>Test Automation"]
JiveGlobals["JiveGlobals<br>Property Management"]
PluginDir["Plugin Directory<br>File Resources"]
ClassLoader["Plugin ClassLoader<br>Isolation"]

PluginManager --> TeamsService
AdminConsole --> WebContext
AuthFilter --> WSContext
TeamsService --> JiveGlobals
WebContext --> PluginDir
WSContext --> ClassLoader

subgraph subGraph2 ["Configuration Sources"]
    JiveGlobals
    PluginDir
    ClassLoader
end

subgraph subGraph1 ["CAS Plugin Components"]
    TeamsService
    WebContext
    WSContext
    TestPlanner
    TestPlanner --> TeamsService
end

subgraph subGraph0 ["Openfire Plugin System"]
    PluginManager
    AdminConsole
    AuthFilter
end
```

**Plugin Lifecycle Management**

The plugin follows standard Openfire plugin patterns with initialization and destruction phases:

| Phase | Method | Purpose |
| --- | --- | --- |
| Initialization | `initializeService()` | Setup services, web contexts, authentication exclusions |
| Runtime | Event processing | Handle calls, messages, user synchronization |
| Destruction | `destroyService()` | Cleanup resources, stop services, remove exclusions |

**Web Context Configuration**

The plugin establishes multiple web contexts:

* **Static Web Content** (`/casweb`): Serves HTML, CSS, JavaScript files
* **WebSocket Endpoints** (`/stream-ws`): Real-time audio streaming and communication
* **Test Interface**: Automated testing and diagnostics

**Authentication Integration**

The plugin modifies Openfire's authentication behavior by adding exclusions to `AuthCheckFilter` for public endpoints while maintaining security for administrative functions.

Sources: [src/java/com/comitfs/openfire/TeamsService.java L305-L381](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TeamsService.java#L305-L381)

 [src/java/com/comitfs/openfire/TeamsService.java L270-L303](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TeamsService.java#L270-L303)

 [src/java/com/comitfs/openfire/TeamsService.java L337-L378](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TeamsService.java#L337-L378)

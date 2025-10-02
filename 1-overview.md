# Overview

> **Relevant source files**
> * [changelog.html](https://github.com/ComitFS/cas-service/blob/b7087e8d/changelog.html)
> * [plugin.xml](https://github.com/ComitFS/cas-service/blob/b7087e8d/plugin.xml)
> * [pom.xml](https://github.com/ComitFS/cas-service/blob/b7087e8d/pom.xml)
> * [readme.html](https://github.com/ComitFS/cas-service/blob/b7087e8d/readme.html)
> * [readme.md](https://github.com/ComitFS/cas-service/blob/b7087e8d/readme.md)
> * [src/java/com/ifsoft/openlink/view/RunTest.java](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/ifsoft/openlink/view/RunTest.java)
> * [src/java/com/ifsoft/openlink/view/SummariseCall.java](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/ifsoft/openlink/view/SummariseCall.java)
> * [src/java/org/ifsoft/upload/Servlet.java](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/org/ifsoft/upload/Servlet.java)
> * [src/web/WEB-INF/web.xml](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/web/WEB-INF/web.xml)

This document provides a comprehensive introduction to the CAS Service Plugin, an Openfire server extension that integrates Microsoft Teams, Azure Communication Services, and automated call testing capabilities. The plugin serves as a communication automation platform that bridges XMPP messaging, Microsoft cloud services, and telephony systems.

For detailed information about specific service components, see [Core Services](./2-core-services.md). For test automation framework details, see [Test Automation Framework](./3-test-automation-framework.md). For client application documentation, see [Client Applications](./5-client-applications.md).

## System Purpose and Architecture

The CAS Service Plugin extends the `RESTServicePlugin` class to provide a unified communication platform with three primary functions:

1. **Microsoft Teams Integration** - Seamless integration with Teams, Graph API, and Bot Framework
2. **Call Automation** - Azure Communication Services integration for automated call testing and IVR validation
3. **Real-time Communication** - WebSocket-based audio streaming and live event processing

## Plugin Architecture Overview

```mermaid
flowchart TD

OF["Openfire XMPP Server"]
Plugin["RESTServicePlugin<br>(casapi)"]
TS["TeamsService"]
CCS["CasCompanionService"]
TP["TestPlanner"]
JW["JerseyWrapper<br>(/v1/*)"]
WSS["WebEventSourceServlet<br>(/sse)"]
TestUI["TestCalls Servlet<br>(/test-calls)"]
SummaryUI["Summary Servlet<br>(/cas-companion-summary)"]
Upload["Upload Servlet<br>(/upload)"]
MSGraph["MsGraphServlet<br>(/notify-call-records)"]
RunTest["RunTest Servlet<br>(/run-test)"]
Teams["Microsoft Teams"]
ACS["Azure Communication Services"]
GraphAPI["Microsoft Graph API"]
AzureAI["Azure AI Services"]

TS --> Teams
TS --> ACS
TS --> GraphAPI
TS --> AzureAI

subgraph subGraph4 ["External Services"]
    Teams
    ACS
    GraphAPI
    AzureAI
end

subgraph subGraph3 ["Openfire Server Environment"]
    OF
    Plugin
    OF --> Plugin
    Plugin --> TS
    Plugin --> CCS
    Plugin --> TP
    JW --> CCS
    WSS --> TS
    TestUI --> TP
    Upload --> TP
    RunTest --> TP

subgraph subGraph2 ["Utility Services"]
    Upload
    MSGraph
    RunTest
end

subgraph subGraph1 ["Web Interface Layer"]
    JW
    WSS
    TestUI
    SummaryUI
end

subgraph subGraph0 ["Core Service Layer"]
    TS
    CCS
    TP
end
end
```

**Sources:** [plugin.xml L4-L30](https://github.com/ComitFS/cas-service/blob/b7087e8d/plugin.xml#L4-L30)

 [src/web/WEB-INF/web.xml L7-L184](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/web/WEB-INF/web.xml#L7-L184)

 [pom.xml L9-L13](https://github.com/ComitFS/cas-service/blob/b7087e8d/pom.xml#L9-L13)

## Maven Project Structure

The plugin is built as a Maven project with comprehensive dependencies for Microsoft services integration:

| Component Category | Key Dependencies | Purpose |
| --- | --- | --- |
| Microsoft Integration | `microsoft-graph`, `msal4j`, `bot-connector` | Teams and Graph API integration |
| Azure Services | `azure-communication-*`, `azure-identity` | Call automation and messaging |
| Web Framework | `jersey-container-*`, `jackson-*` | REST API and JSON processing |
| Test Automation | `quartz`, `nashorn-core`, `opencsv` | Scheduled testing and script execution |
| Real-time Communication | `netty-all`, `webrtc-java` | WebSocket and WebRTC support |

**Sources:** [pom.xml L47-L340](https://github.com/ComitFS/cas-service/blob/b7087e8d/pom.xml#L47-L340)

## Service Integration Flow

```mermaid
sequenceDiagram
  participant External Client
  participant JerseyWrapper
  participant CasCompanionService
  participant TeamsService
  participant Azure Communication Services
  participant WebEventSourceServlet
  participant Teams

  External Client->>JerseyWrapper: "HTTP Request (/v1/*)"
  JerseyWrapper->>CasCompanionService: "Route to REST endpoint"
  CasCompanionService->>CasCompanionService: "ensureUserExists()"
  CasCompanionService->>TeamsService: "Delegate business logic"
  loop [Microsoft Teams Operations]
    TeamsService->>Teams: "Graph API calls"
    Teams-->>TeamsService: "Response data"
    TeamsService->>Azure Communication Services: "Call automation API"
    Azure Communication Services-->>TeamsService: "Call events"
  end
  TeamsService->>WebEventSourceServlet: "sendPayload() for real-time updates"
  WebEventSourceServlet->>External Client: "Server-sent events"
  TeamsService-->>CasCompanionService: "Processed response"
  CasCompanionService-->>JerseyWrapper: "JSON response"
  JerseyWrapper-->>External Client: "HTTP response"
```

**Sources:** [src/web/WEB-INF/web.xml L106-L108](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/web/WEB-INF/web.xml#L106-L108)

 [src/web/WEB-INF/web.xml L32-L35](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/web/WEB-INF/web.xml#L32-L35)

## Core Capabilities

### Microsoft Teams Integration

The plugin provides deep integration with Microsoft Teams ecosystem through:

* **Teams App Support** - Custom Teams application with multiple tabs and bot functionality
* **Graph API Integration** - Access to user profiles, meetings, and presence information
* **Authentication** - MSAL-based OAuth integration for secure access

### Call Automation Framework

Comprehensive automated testing capabilities include:

* **Test Scenario Management** - CSV and JavaScript-based test definitions
* **Scheduled Execution** - Quartz-based job scheduling for automated test runs
* **IVR Testing** - Automated interaction with phone systems using DTMF and voice recognition
* **Results Reporting** - HTML and JSON output formats for test results

### Real-time Communication

Advanced real-time features support:

* **Audio Streaming** - WebSocket-based live audio processing
* **Speech-to-Text** - Azure AI integration for real-time transcription
* **Event Broadcasting** - Server-sent events for live system updates

**Sources:** [pom.xml L138-L265](https://github.com/ComitFS/cas-service/blob/b7087e8d/pom.xml#L138-L265)

 [src/java/com/ifsoft/openlink/view/SummariseCall.java L64-L70](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/ifsoft/openlink/view/SummariseCall.java#L64-L70)

## File System Organization

```mermaid
flowchart TD

PluginXml["plugin.xml"]
PomXml["pom.xml"]
Readme["readme.md"]
Changelog["changelog.html"]
WebInf["WEB-INF/web.xml"]
SrcJava["src/java/"]
Openfire["org.jivesoftware.openfire.plugin.rest"]
Ifsoft["org.ifsoft.*"]
Comitfs["com.comitfs.openfire"]
Upload["org.ifsoft.upload"]
SrcWeb["src/web/"]
JSP["*.jsp files"]
Static["Static resources"]
Tests["tests/ directory"]

subgraph subGraph3 ["Configuration Files"]
    PluginXml
    PomXml
    Readme
    Changelog
end

subgraph subGraph2 ["Source Structure"]
    SrcJava
    SrcWeb
    SrcJava --> Openfire
    SrcJava --> Ifsoft
    SrcJava --> Comitfs
    SrcJava --> Upload
    SrcWeb --> WebInf
    SrcWeb --> JSP
    SrcWeb --> Static
    SrcWeb --> Tests

subgraph subGraph1 ["Web Resources"]
    WebInf
    JSP
    Static
    Tests
end

subgraph subGraph0 ["Java Packages"]
    Openfire
    Ifsoft
    Comitfs
    Upload
end
end
```

**Sources:** [src/java/org/ifsoft/upload/Servlet.java L1](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/org/ifsoft/upload/Servlet.java#L1-L1)

 [src/java/com/ifsoft/openlink/view/SummariseCall.java L1](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/ifsoft/openlink/view/SummariseCall.java#L1-L1)

 [src/web/WEB-INF/web.xml L1](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/web/WEB-INF/web.xml#L1-L1)

## Admin Console Integration

The plugin extends Openfire's admin console with dedicated management interfaces organized into three main areas:

* **Companion Service** - System overview and REST API settings
* **Teams Integration** - User profiles, settings, and call history management
* **Call Automation** - Test management and call summarization tools

These interfaces are accessible through the admin console at `/cas-companion-summary`, `/teams-settings`, and `/test-calls` respectively.

**Sources:** [plugin.xml L13-L29](https://github.com/ComitFS/cas-service/blob/b7087e8d/plugin.xml#L13-L29)

 [src/web/WEB-INF/web.xml L125-L183](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/web/WEB-INF/web.xml#L125-L183)
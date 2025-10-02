# Test Automation Framework

> **Relevant source files**
> * [src/java/com/comitfs/openfire/TestPlanner.java](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TestPlanner.java)
> * [src/web/javascripts/testcalls.js](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/web/javascripts/testcalls.js)

The Test Automation Framework provides comprehensive automated testing capabilities for call flows, IVR systems, and communication workflows. This framework enables both CSV-driven scenario testing and JavaScript-based custom test execution, with integrated scheduling, real-time monitoring, and detailed reporting capabilities.

For information about the core TestPlanner engine implementation, see [TestPlanner Engine](./3.1-testplanner-engine.md). For details about creating test scenarios, see [Test Scenario Definitions](./3.2-test-scenario-definitions.md). For test results and reporting formats, see [Test Results & Reporting](./3.3-test-results-and-reporting.md). For the web-based management interface, see [Test Management Web Interface](./3.4-test-management-web-interface.md).

## Architecture Overview

The Test Automation Framework is built around the `TestPlanner` class which serves as the central orchestrator for all testing activities. It integrates with Azure Communication Services for call automation, Quartz Scheduler for job management, and provides both CSV and JavaScript-based test execution engines.

```mermaid
flowchart TD

TP["TestPlanner"]
QS["Quartz Scheduler"]
NE["Nashorn JavaScript Engine"]
CSV["CSV Scenarios<br>scenario1.csv"]
JS["JavaScript Tests<br>test1.js"]
Media["Test Media Files"]
TS["TeamsService<br>Call Automation"]
WS["WebSocket Streaming<br>Real-time Events"]
Transcript["Transcript Collection<br>ConcurrentLinkedQueue"]
HTML["HTML Reports<br>filename.csv.html"]
JSON["JSON Data<br>filename.csv.json"]
Logs["System Logs"]
TestUI["testcalls.js<br>Web Interface"]
Upload["File Upload<br>Test Assets"]
Monitor["Real-time Monitoring<br>WebSocket Events"]

TP --> CSV
TP --> JS
TP --> TS
TP --> WS
TP --> Transcript
TP --> HTML
TP --> JSON
TestUI --> TP

subgraph subGraph4 ["Web Interface"]
    TestUI
    Upload
    Monitor
    TestUI --> Upload
    TestUI --> Monitor
end

subgraph subGraph3 ["Results Generation"]
    HTML
    JSON
    Logs
end

subgraph subGraph2 ["Execution Environment"]
    TS
    WS
    Transcript
end

subgraph subGraph1 ["Test Definitions"]
    CSV
    JS
    Media
    CSV --> Media
    JS --> Media
end

subgraph subGraph0 ["Test Execution Layer"]
    TP
    QS
    NE
    TP --> QS
    TP --> NE
end
```

**Sources:** [src/java/com/comitfs/openfire/TestPlanner.java L1-L653](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TestPlanner.java#L1-L653)

 [src/web/javascripts/testcalls.js L1-L208](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/web/javascripts/testcalls.js#L1-L208)

## Core Components

### TestPlanner Class Structure

The `TestPlanner` class implements multiple interfaces and provides comprehensive test automation capabilities:

| Component | Purpose | Implementation |
| --- | --- | --- |
| **Job Interface** | Quartz scheduler integration | `TestPlanner implements Job` |
| **Script Engine** | JavaScript test execution | Nashorn engine initialization |
| **Transcript Management** | Real-time transcription collection | `ConcurrentLinkedQueue<String>` |
| **Cron Job Management** | Scheduled test execution | Quartz `CronTrigger` configuration |
| **Report Generation** | HTML and JSON output | File-based result persistence |

```mermaid
flowchart TD

Init["initialize()<br>Line 52"]
Destroy["destroy()<br>Line 107"]
Execute["execute()<br>Line 589"]
ProcessTest["processTest()<br>Line 119"]
ProcessTrigger["processTestTrigger()<br>Line 127"]
CSVTest["testCSVFile()<br>Line 156"]
JSTest["testJSFile()<br>Line 142"]
CreateHTML["createHTMLFile()<br>Line 354"]
StartCall["startTestCall()<br>Line 455"]
StopCall["stopTestCall()<br>Line 475"]
SendVoice["sendVoiceCommand()<br>Line 483"]
SendDTMF["sendDTMF()<br>Line 492"]

Init --> ProcessTest
CSVTest --> CreateHTML
JSTest --> CreateHTML
CSVTest --> StartCall
CSVTest --> StopCall
CSVTest --> SendVoice
CSVTest --> SendDTMF

subgraph subGraph2 ["Utility Functions"]
    CreateHTML
    StartCall
    StopCall
    SendVoice
    SendDTMF
end

subgraph subGraph1 ["Test Processing"]
    ProcessTest
    ProcessTrigger
    CSVTest
    JSTest
    ProcessTrigger --> CSVTest
    ProcessTrigger --> JSTest
end

subgraph subGraph0 ["TestPlanner Core"]
    Init
    Destroy
    Execute
end
```

**Sources:** [src/java/com/comitfs/openfire/TestPlanner.java L41-L653](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TestPlanner.java#L41-L653)

### JavaScript Engine Integration

The framework uses the Nashorn JavaScript engine to execute custom test scripts with direct access to TestPlanner functionality:

```mermaid
flowchart TD

Factory["ScriptEngineManager<br>Line 55"]
Engine["Nashorn Engine<br>Line 56"]
Eval["Script Evaluation<br>Line 149"]
Debug["debug(data)"]
Info["info(data)"]
Wait["wait(secs)"]
StartTestCall["startTestCall(phone, callerID)"]
StopTestCall["stopTestCall(serverCallId)"]
SendVoiceCommand["sendVoiceCommand(callId, text, phone, options)"]
SendDTMF["sendDTMF(callId, dtmfs, phone)"]
GetTranscript["getTranscript(serverCallId)"]

Eval --> Debug
Eval --> Info
Eval --> Wait
Eval --> StartTestCall
Eval --> StopTestCall
Eval --> SendVoiceCommand
Eval --> SendDTMF
Eval --> GetTranscript

subgraph subGraph1 ["Exposed API Functions"]
    Debug
    Info
    Wait
    StartTestCall
    StopTestCall
    SendVoiceCommand
    SendDTMF
    GetTranscript
end

subgraph subGraph0 ["JavaScript Test Environment"]
    Factory
    Engine
    Eval
    Factory --> Engine
    Engine --> Eval
end
```

**Sources:** [src/java/com/comitfs/openfire/TestPlanner.java L55-L57](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TestPlanner.java#L55-L57)

 [src/java/com/comitfs/openfire/TestPlanner.java L465-L515](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TestPlanner.java#L465-L515)

## Test Scenario Types

### CSV-Based Test Scenarios

CSV scenarios provide structured, data-driven test execution with predefined columns for test steps:

| Column | Purpose | Example Values |
| --- | --- | --- |
| **Prompt** | Expected audio response | "Press 1 for sales" |
| **Timeout** | Wait duration (seconds) | "5", "10" |
| **Action** | Test operation | "start", "dtmf", "voice", "stop" |
| **Value1** | Primary parameter | Phone number, DTMF digits, text |
| **Value2** | Secondary parameter | Caller ID, voice options |

The CSV test execution follows this process flow:

```mermaid
sequenceDiagram
  participant TestPlanner
  participant CSV Reader
  participant TeamsService
  participant Transcript Queue
  participant HTML/JSON Reports

  TestPlanner->>CSV Reader: "readNext() - Line 177"
  CSV Reader-->>TestPlanner: "Step parameters [prompt, timeout, action, value1, value2]"
  loop ["Action: start/begin"]
    TestPlanner->>TeamsService: "startTestCall(phoneNumber, callerID)"
    TeamsService-->>TestPlanner: "serverCallId"
    TestPlanner->>Transcript Queue: "Create transcript queue"
    TestPlanner->>TeamsService: "sendDTMF(serverCallId, dtmfs, phoneNumber)"
    TestPlanner->>TeamsService: "sendVoiceCommand(serverCallId, text, phoneNumber, options)"
    TestPlanner->>TeamsService: "stopTestCall(serverCallId)"
    TestPlanner->>TestPlanner: "Wait 10 seconds for final transcript"
  end
  TestPlanner->>HTML/JSON Reports: "createHTMLFile() - Generate results"
  TestPlanner->>HTML/JSON Reports: "Generate JSON data file"
```

**Sources:** [src/java/com/comitfs/openfire/TestPlanner.java L156-L352](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TestPlanner.java#L156-L352)

### JavaScript Test Scripts

JavaScript tests provide full programmatic control with access to all TestPlanner API functions:

```mermaid
flowchart TD

LoadScript["Load .js file<br>Line 146-147"]
ParseScript["Parse script content<br>Line 148"]
ExecuteScript["engine.eval(script)<br>Line 149"]
CallMethods["Call Control:<br>startTestCall(), stopTestCall()"]
DTMFMethods["DTMF Control:<br>sendDTMF()"]
VoiceMethods["Voice Control:<br>sendVoiceCommand()"]
UtilityMethods["Utilities:<br>wait(), debug(), info(), error()"]
TranscriptMethods["Transcript Access:<br>getTranscript()"]

ExecuteScript --> CallMethods
ExecuteScript --> DTMFMethods
ExecuteScript --> VoiceMethods
ExecuteScript --> UtilityMethods
ExecuteScript --> TranscriptMethods

subgraph subGraph1 ["Available API Methods"]
    CallMethods
    DTMFMethods
    VoiceMethods
    UtilityMethods
    TranscriptMethods
end

subgraph subGraph0 ["JavaScript Test Flow"]
    LoadScript
    ParseScript
    ExecuteScript
    LoadScript --> ParseScript
    ParseScript --> ExecuteScript
end
```

**Sources:** [src/java/com/comitfs/openfire/TestPlanner.java L142-L154](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TestPlanner.java#L142-L154)

 [src/java/com/comitfs/openfire/TestPlanner.java L465-L515](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TestPlanner.java#L465-L515)

## Scheduling and Job Management

### Quartz Integration

The framework uses Quartz Scheduler for automated test execution with cron-based scheduling:

```mermaid
flowchart TD

Init["initialize()<br>Line 52"]
Factory["StdSchedulerFactory.getDefaultScheduler()<br>Line 66"]
Start["scheduler.start()<br>Line 67"]
LoadProperties["JiveGlobals.getPropertyNames()<br>Line 69"]
FilterCron["Filter 'cas.cron.' properties<br>Line 74"]
ProcessCron["processTestCron()<br>Line 81"]
JobDetail["newJob(TestPlanner.class)<br>Line 640"]
CronTrigger["cronSchedule(specTrigger)<br>Line 641"]
ScheduleJob["scheduler.scheduleJob()<br>Line 643"]
ExecuteJob["execute() - Job execution<br>Line 589"]

Start --> LoadProperties
ProcessCron --> JobDetail

subgraph subGraph2 ["Job Execution"]
    JobDetail
    CronTrigger
    ScheduleJob
    ExecuteJob
    JobDetail --> CronTrigger
    CronTrigger --> ScheduleJob
    ScheduleJob --> ExecuteJob
end

subgraph subGraph1 ["Cron Job Processing"]
    LoadProperties
    FilterCron
    ProcessCron
    LoadProperties --> FilterCron
    FilterCron --> ProcessCron
end

subgraph subGraph0 ["Scheduler Initialization"]
    Init
    Factory
    Start
    Init --> Factory
    Factory --> Start
end
```

**Sources:** [src/java/com/comitfs/openfire/TestPlanner.java L52-L100](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TestPlanner.java#L52-L100)

 [src/java/com/comitfs/openfire/TestPlanner.java L602-L651](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TestPlanner.java#L602-L651)

### Job Configuration Properties

Cron jobs are configured through Openfire properties with the pattern `cas.cron.<key>`:

| Property Pattern | Purpose | Example |
| --- | --- | --- |
| `cas.cron.{key}` | Cron schedule expression | `0 0 8 * * ?` (daily at 8 AM) |
| Job key | Test file identifier | `scenario1.csv`, `test1.js` |
| Trigger parsing | Optional title extraction | `0 0 8 * * ?(Daily Test)` |

**Sources:** [src/java/com/comitfs/openfire/TestPlanner.java L74-L88](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TestPlanner.java#L74-L88)

 [src/java/com/comitfs/openfire/TestPlanner.java L630-L638](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TestPlanner.java#L630-L638)

## Real-time Communication Integration

### WebSocket Integration

The framework integrates with the live streaming system for real-time test monitoring and audio processing:

```mermaid
flowchart TD

TestUI["testcalls.js"]
WebSocket["WebSocket /stream-ws/"]
AudioProcessor["Audio Worklet Processor"]
LSS["LiveStreamSocket"]
TP["TestPlanner"]
Transcripts["Transcript Queues"]
TS["TeamsService"]
ACS["Azure Communication Services"]

WebSocket --> LSS
TP --> TS

subgraph subGraph2 ["Call Automation"]
    TS
    ACS
    TS --> ACS
end

subgraph subGraph1 ["Server Components"]
    LSS
    TP
    Transcripts
    LSS --> TP
    TP --> Transcripts
end

subgraph subGraph0 ["Web Interface"]
    TestUI
    WebSocket
    AudioProcessor
    TestUI --> WebSocket
    WebSocket --> AudioProcessor
    AudioProcessor --> TestUI
end
```

**Sources:** [src/web/javascripts/testcalls.js L68-L163](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/web/javascripts/testcalls.js#L68-L163)

 [src/java/com/comitfs/openfire/TestPlanner.java L44-L45](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TestPlanner.java#L44-L45)

### Transcript Collection

Real-time transcription data is collected during test execution using concurrent data structures:

```mermaid
flowchart TD

Create["Create queue on call start<br>Line 219, 225"]
Collect["Collect transcript lines during call"]
Process["Process transcripts in test validation<br>Line 282-297"]
Cleanup["Clear and remove on test completion<br>Line 444-446"]
TranscriptMap["Map> transcripts<br>Line 45"]
ServerCallId["serverCallId - Call identifier"]
Queue["ConcurrentLinkedQueue - Per-call transcript"]

subgraph subGraph1 ["Transcript Operations"]
    Create
    Collect
    Process
    Cleanup
    Create --> Collect
    Collect --> Process
    Process --> Cleanup
end

subgraph subGraph0 ["Transcript Management"]
    TranscriptMap
    ServerCallId
    Queue
    TranscriptMap --> ServerCallId
    ServerCallId --> Queue
end
```

**Sources:** [src/java/com/comitfs/openfire/TestPlanner.java L45](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TestPlanner.java#L45-L45)

 [src/java/com/comitfs/openfire/TestPlanner.java L219-L226](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TestPlanner.java#L219-L226)

 [src/java/com/comitfs/openfire/TestPlanner.java L282-L297](https://github.com/ComitFS/cas-service/blob/b7087e8d/src/java/com/comitfs/openfire/TestPlanner.java#L282-L297)
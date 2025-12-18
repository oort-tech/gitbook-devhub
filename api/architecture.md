# System Architecture & Workflow



## Task Status Definitions
The lifecycle of a task is tracked using the following status codes:

| Status | Description |
| :--- | :--- |
| **0** | **Initial:** Task created but not started. |
| **1** | **Running:** Task is in progress. |
| **2** | **Paused:** Task is paused, waiting to resume. |
| **3** | **Completed:** Task successfully finished. |
| **4** | **Cancelled:** Task has been cancelled. |
| **5** | **Failed:** Task execution failed. |

## 1. Normal Execution Flow
The following sequence describes the standard lifecycle of a task, from creation to result upload.

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant FE as Datahub FE / Plugin JS
    participant DH as Datahub Backend
    participant PL as Plugin Backend
    participant S3 as 3rd Party Storage

    %% Initialization
    User->>FE: Create Task / Select Plugin
    FE->>FE: Auto-load Plugin JS
    FE->>User: Pop up Config Page
    User->>FE: Fill Params & Submit
    FE->>DH: Save Task Config
    DH->>PL: POST /api/plugin/create_task
    PL->>DH: Response {code: 0, msg: "success"}

    %% Execution
    PL->>DH: POST /api/task/update_status (Status: 1 RUNNING)
    
    opt Optional Download
        PL->>DH: POST /api/file/download_presign
        DH->>PL: Return input_file_url (Presigned)
        PL->>S3: GET {input_file_url}
        S3->>PL: Return File Stream
    end

    par Parallel: Heartbeat & Core Logic
        loop Heartbeat every 30s
            PL->>DH: POST /api/task/heartbeat
        end
        PL->>PL: Core Business Logic (Crawl/Process/Package)
    end

    %% Upload
    PL->>DH: POST /api/task/upload/init
    DH->>PL: Return upload_id & Part Presigned URLs
    loop Loop all parts
        PL->>S3: PUT {part_url} (Direct Upload)
        S3->>PL: Return ETag
    end
    PL->>DH: POST /api/task/upload/complete
    DH->>PL: Return file_url

    %% Completion
    PL->>DH: POST /api/task/result
    DH->>DH: Update final status & result
    DH->>PL: Response {code: 0}
```  

## 2. Cancellation Flow
When a user cancels a task, Datahub notifies the plugin to terminate the process. 

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant DH as Datahub Backend
    participant PL as Plugin Backend

    User->>DH: Click "Cancel Task"
    DH->>PL: POST /api/plugin/manage_task (Status: 4)
    PL->>PL: Terminate running process
    PL->>DH: POST /api/task/update_status (Status: 4 CANCELLED)
    DH->>PL: Response {code: 0}
```
    
## 3. Pause & Resume Flow
Datahub can request a plugin to suspend and resume operations.

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant DH as Datahub Backend
    participant PL as Plugin Backend

    %% Pause
    User->>DH: Request "Pause Task"
    DH->>PL: POST /api/plugin/manage_task (Status: 2)
    PL->>PL: Suspend running process
    PL->>DH: POST /api/task/update_status (Status: 2 PAUSED)
    DH->>PL: Response {code: 0}

    %% Resume
    User->>DH: Request "Resume Task"
    DH->>PL: POST /api/plugin/manage_task (Status: 1)
    PL->>PL: Resume process execution
    PL->>DH: POST /api/task/update_status (Status: 1 RUNNING)
    DH->>PL: Response {code: 0}
```
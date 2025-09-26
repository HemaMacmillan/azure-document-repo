# Upload Workflow Sequence Diagram

```mermaid
sequenceDiagram
    participant U as User (Editor/Admin)
    participant UI as Web App
    participant API as Backend API (App Service)
    participant DEF as Defender for Storage
    participant BLOB as Azure Blob Storage
    participant SQL as Azure SQL DB
    participant AUD as Audit Log

    U->>UI: Select file + metadata
    UI->>API: Submit upload request
    API->>DEF: Trigger malware scan
    DEF-->>API: Scan result (clean/malicious)

    alt Malicious
        API-->>U: Reject upload + notify Admin
    else Clean
        API->>BLOB: Store file in /files container
        API->>SQL: Insert metadata + ownership info
        API->>AUD: Record "Upload" action
        API-->>U: Confirm success + return file ID
    end

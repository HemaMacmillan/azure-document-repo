# Purge Workflow Sequence Diagram

sequenceDiagram
    participant U as User (Editor/Admin)
    participant API as Backend API
    participant BLOB as Azure Blob Storage
    participant SQL as Azure SQL DB
    participant AUD as Audit Log
    participant FN as Nightly Function
    participant MAIL as Email Service (Logic App)

    U->>API: Soft delete file (own or admin)
    API->>BLOB: Mark blob soft-deleted (30d retention)
    API->>SQL: Update IsSoftDeleted flag + SoftDeletedUtc
    API->>AUD: Record "SoftDelete" action

    loop Nightly
        FN->>BLOB: Scan for soft-deleted files
        FN->>SQL: Fetch metadata & retention status
        alt File ≤ 10 days before purge
            FN->>MAIL: Send purge warning email (creator + updaters, cc admins)
            MAIL-->>U: Purge warning received
        end
        alt File ≥ 30 days expired
            FN->>BLOB: Permanently delete file
            FN->>SQL: Update records (purged)
            FN->>AUD: Record "PermanentDelete"
        end
    end

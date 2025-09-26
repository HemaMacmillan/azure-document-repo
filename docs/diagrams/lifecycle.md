# Lifecycle & Purge Flow

```mermaid
flowchart TD
  U[Upload File] --> V{Validate size/type?}
  V -- No --> E1[Reject upload]
  V -- Yes --> D[Defender for Storage scan]

  D -- Malicious --> Q[Quarantine + Notify Admin]
  D -- Clean --> B[Store in Blob (Hot tier)]
  B --> M[Metadata in SQL + Audit log]

  %% Lifecycle progression
  M --> L1[Blob Lifecycle Policy]
  L1 -->|≥30 days idle| C[Cool tier]
  C -->|≥90 days idle| CO[Cold tier]

  %% Soft delete & purge
  DEL[Soft Delete Action] --> SD[Mark as soft deleted (30d retention)]
  SD --> PL[Purge List UI Page]

  %% Nightly Function
  NF[Azure Function - Nightly Job] --> SCAN[Scan storage for purge candidates]
  SCAN -->|≤10 days to purge| EMAIL[Email Creator + Updaters (cc Admins)]
  SCAN --> PL

  %% Final purge
  SD -->|30d expired| PURGE[Permanent Delete]

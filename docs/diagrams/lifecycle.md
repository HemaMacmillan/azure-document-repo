# Lifecycle & Purge Flow

  U[Upload File] --> V{Valid size/type?}
  V -- No --> R[Reject Upload]
  V -- Yes --> S[Defender Scan]

  S -- Malicious --> Q[Quarantine + Notify Admin]
  S -- Clean --> B[Store in Blob (Hot Tier)]
  B --> M[Write Metadata + Audit in SQL]

  M --> L[Lifecycle Policy]
  L -->|>= 30 days idle| C[Move to Cool Tier]
  C -->|>= 90 days idle| CL[Move to Cold Tier]

  D[Soft Delete Action] --> SD[Mark Soft-Deleted (30d Retention)]
  SD --> PL[Purge List UI Page]

  NF[Nightly Function] --> SC[Scan for Purge Candidates]
  SC -->|<= 10 days before purge| E[Email Creator + Updaters (cc Admins)]
  SC --> PL

  SD -->|30 days expired| P[Permanent Delete]

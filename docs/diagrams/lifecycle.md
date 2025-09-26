# Lifecycle & Purge Flow

  U[Upload File] --> V{Validate size/type?}
  V -- No --> E1[Reject upload]
  V -- Yes --> D[Defender scan]
  D -- Malicious --> Q[Quarantine + notify]
  D -- Clean --> B[Store in Blob (Hot)]
  B --> M[Write SQL metadata + audit]
  M --> L1[Lifecycle policy]
  L1 -->|>= 30d idle| C[Cool tier]
  C -->|>= 90d idle| CL[Cold tier]
  DEL[Soft delete action] --> SD[Soft-deleted (30d retention)]
  SD --> PL[Purge List UI]
  NF[Nightly Function] --> SCAN[Scan for purge candidates]
  SCAN -->|<= 10d to purge| EMAIL[Email creator + updaters (cc Admins)]
  SCAN --> PL
  SD -->|30d expired| PURGE[Permanent delete]

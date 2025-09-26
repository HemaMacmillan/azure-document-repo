Azure Document Repository
Design and architecture for a file management system built on Azure services. This repo contains diagrams, requirements, and backlog issues to showcase a cloud-native document repository migration from legacy systems.
________________________________________
🌐 Overview
•	File types: PDF, Word (.docx)
•	Max size: 1 MB per file
•	Features: Upload, download, preview (in-app), edit metadata, search, category tree navigation
•	Search: Wildcard on filename & tags; category/subcategory browsing
•	IAM Roles:
o	Admin → full access
o	Editor → limited to own files (upload, re-upload, metadata edit, delete/restore, lock/unlock)
o	User → view + search only
•	Platform: Azure Blob Storage + SQL Database + App Service + Functions + Key Vault
•	Soft delete: 30 days retention with restore option
•	Notifications: Email alerts 10 days before purge + purge list page
________________________________________
🏗 Architecture Diagram
flowchart TB
  subgraph Client
    UI[Web App (static/front-end)]
  end

  subgraph Azure
    APIM[API Management]
    BE[Backend API (App Service)]
    SQL[(Azure SQL DB\nMetadata + Audit + IAM + Category Master)]
    KV[Key Vault]

    subgraph Storage
      SA[(Storage Account)]
      Blob[Blob Container: /files]
      Logs[Blob Container: /audit-exports]
    end

    MSI[(Managed Identity)]
    Def[Defender for Storage\nMalware scanning]
    Fn[Azure Functions\n- Nightly lifecycle + notifications\n- On‑upload orchestration]
    LA[Logic App (email notifications)]
  end

  UI -->|Auth (Entra ID)| APIM --> BE
  BE -->|Read/Write| SQL
  BE -->|SAS / SDK| Blob
  BE --> KV
  Def --> SA
  Fn --> Blob
  Fn --> SQL
  Fn --> LA
  UI -->|Queries| APIM --> BE -->|Search| SQL
________________________________________
🗂 Data Model
•	Files: File metadata, blob path, created/updated by, soft delete flag
•	Metadata: Author, tags
•	Categories / SubCategories: Static master data
•	Users / Roles / UserRoles: IAM mapping
•	Audit: Tracks upload, edit, delete, restore, lock/unlock, download, view actions
________________________________________
🔑 IAM Matrix
Capability	Admin	Editor	User
Browse/Search	✓	✓	✓
Preview	✓	✓	✓
Download	✓	✓	✗
Upload	✓	✓	✗
Re-upload	✓ (any)	✓ (own)	✗
Edit metadata	✓ (any)	✓ (own)	✗
Delete	✓ (any)	✓ (own, soft delete)	✗
Restore	✓ (any)	✓ (own)	✗
Lock/Unlock	✓ (any)	✓ (own)	✗
________________________________________
🔄 Workflows
•	Upload → Validate size/type → Defender scan → Store in Blob + Metadata in SQL → Audit
•	Preview → PDF native, DOCX converted to PDF
•	Soft delete → Flag + Blob soft delete → Restorable up to 30 days
•	Purge list → UI page shows files pending purge with metadata
•	Nightly job → Lifecycle tiering + email alerts (10 days before purge)
________________________________________
📊 Storage & Lifecycle
•	Access tiers: Hot → Cool (≥30d) → Cold (≥90d)
•	Soft delete: 30 days
•	Versioning: Disabled for MVP
•	Encryption: Enabled (MMK, CMK optional)
________________________________________
📌 Backlog (Issues to Create)
•	feat(storage): enable soft delete (30d), lifecycle Hot→Cool→Cold
•	feat(api): roles & policies (Admin/Editor/User)
•	feat(api): upload validation & audit logging
•	feat(api): blob lease lock/unlock
•	feat(ui): category tree & counts
•	feat(ui): search (filename/tags wildcard) + filters
•	feat(ui): PDF preview; DOCX→PDF preview
•	feat(ui): purge list page (30‑day soft‑deleted files)
•	feat(job): nightly purge-warning email (10d)
•	feat(security): Defender for Storage malware scanning
•	docs: diagrams (context, ERD, sequences, IAM, purge list)
________________________________________
📁 Repo Structure
/README.md          ← Overview (this file)
/docs               ← Documentation & diagrams
/docs/diagrams      ← PNG/SVG exports of Mermaid diagrams
/backend            ← Placeholder for API code (future)
/infra              ← Infra-as-code (Bicep/ARM templates)
________________________________________

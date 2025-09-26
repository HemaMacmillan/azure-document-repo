
# Entity Relationship Diagram

erDiagram
    USERS {
        int UserId PK
        string Upn
        string DisplayName
    }

    ROLES {
        int RoleId PK
        string Name
    }

    USERROLES {
        int UserId FK
        int RoleId FK
    }

    FILES {
        int FileId PK
        string BlobPath
        string FileName
        int CategoryId FK
        int SubCategoryId FK
        int CreatedByUserId FK
        datetime CreatedUtc
        datetime UpdatedUtc
        boolean IsSoftDeleted
        datetime SoftDeletedUtc
        boolean IsLocked
    }

    METADATA {
        int FileId PK, FK
        string Author
        string Tags
    }

    CATEGORIES {
        int CategoryId PK
        string Name
    }

    SUBCATEGORIES {
        int SubCategoryId PK
        int CategoryId FK
        string Name
    }

    AUDIT {
        int AuditId PK
        int FileId FK
        string Action
        int ActorUserId FK
        datetime TimestampUtc
        string Details
    }

    USERS ||--o{ USERROLES : has
    ROLES ||--o{ USERROLES : assigned
    USERS ||--o{ FILES : creates
    FILES ||--|| METADATA : contains
    CATEGORIES ||--o{ SUBCATEGORIES : groups
    CATEGORIES ||--o{ FILES : categorizes
    SUBCATEGORIES ||--o{ FILES : categorizes
    FILES ||--o{ AUDIT : logs
    USERS ||--o{ AUDIT : performs

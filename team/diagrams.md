# Diagrams — Manager Library

Diagrams of the system as built (Laravel + React), following the spec. GitHub renders the Mermaid blocks below.

## 1. System overview

```mermaid
flowchart LR
    page["page: static presentation site"]
    app["app: React frontend"] -->|"HTTP JSON, Bearer token"| api["api: Laravel backend"]
    tool["tool: demo seeder (not built)"] -.->|"would use the API"| api
    api --> db[("PostgreSQL 16")]
    infra["infra: Docker, empty database"] --> db
```

The `page` repo is independent: it never calls the API. The schema belongs to the Laravel migrations; `infra` only starts an empty engine.

## 2. Data model (six tables)

```mermaid
erDiagram
    category ||--o{ book : classifies
    member ||--o{ loan : requests
    user ||--o{ loan : registers
    loan ||--|{ loan_item : composes
    book ||--o{ loan_item : "lent in (FK RESTRICT)"

    category {
        uuid id PK
        string name
    }
    book {
        uuid id PK
        string title
        string author
        string isbn
        uuid category_id FK
        int total_copies
        int available_copies
        string cover_key
        timestamptz deleted_at
    }
    member {
        uuid id PK
        string full_name
        string document_number
    }
    user {
        uuid id PK
        string username
        string password_hash
        string role
    }
    loan {
        uuid id PK
        uuid member_id FK
        uuid loaned_by_user_id FK
        timestamptz loaned_at
    }
    loan_item {
        uuid id PK
        uuid loan_id FK
        uuid book_id FK
        string book_title
        string isbn
        string category_name
        timestamptz due_at
        timestamptz returned_at
    }
```

`book_title`, `isbn` and `category_name` in `loan_item` are frozen at loan time (RN-4) and do not follow the catalog. The loan status is calculated, never stored.

## 3. Sequence: log in

```mermaid
sequenceDiagram
    participant C as Client
    participant H as HTTP controller
    participant S as Login service
    participant U as User repository
    C->>H: POST /api/v1/auth/login
    H->>S: username and password
    S->>U: find user by lowercased username
    U-->>S: user with password hash
    S->>S: verify Argon2id hash
    alt valid credentials
        S-->>H: JWT valid 8 hours
        H-->>C: 200 access_token, token_type, expires_in, role
    else invalid credentials
        S-->>H: invalid credentials
        H-->>C: 401 INVALID_CREDENTIALS (does not say which field failed)
    end
```

## 4. Sequence: register a loan

```mermaid
sequenceDiagram
    participant C as Client
    participant H as HTTP controller
    participant S as Lend service
    participant M as Member repository
    participant B as Book repository
    participant L as Loan repository
    C->>H: POST /api/v1/loans with member and 1 to 3 books
    H->>S: lend
    S->>S: validate shape and repeated books (422)
    S->>M: get member and unreturned lines
    S->>S: member has overdue books? (409)
    S->>S: unreturned plus requested greater than 3? (422)
    S->>B: get books in one query
    S->>L: save loan and lines in one transaction
    L->>B: conditional update, decrease available copies if any left
    alt a copy was available for every line
        L-->>S: saved, due date is loaned_at plus 14 days
        S-->>H: loan with frozen title, ISBN and category
        H-->>C: 201
    else last copy taken by someone else
        L-->>S: zero rows updated, transaction rolled back
        S-->>H: BOOK_NOT_AVAILABLE
        H-->>C: 409 and nothing is saved
    end
```

The order of the checks is decision 5 in `spec-gaps-and-decisions.md`; the conditional update is decision 1.

## 5. Sequence: return one line

```mermaid
sequenceDiagram
    participant C as Client
    participant H as HTTP controller
    participant S as Return service
    participant L as Loan repository
    participant B as Book repository
    C->>H: POST /api/v1/loans/{loan_id}/items/{item_id}/return
    H->>S: return item
    S->>L: get loan line
    alt line already returned
        S-->>H: ITEM_ALREADY_RETURNED
        H-->>C: 409 (never adds a copy twice)
    else line pending
        S->>L: set returned_at
        S->>B: add one available copy
        S-->>H: updated line
        H-->>C: 200
    end
```

## 6. Loan status (calculated, not stored)

```mermaid
stateDiagram-v2
    [*] --> open: loan registered
    open --> overdue: due date passes with an unreturned line
    open --> closed: all lines returned
    overdue --> closed: all lines returned
    closed --> [*]
```

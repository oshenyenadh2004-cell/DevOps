# Architecture Diagram — NodeGoat DevSecOps Pipeline

## Components
- **Web App (Node.js / Express)** — handles routing, authentication, sessions, and business logic (allocations, profile, contributions)
- **MongoDB** — stores user accounts, sessions, and financial allocation data

## Data Flow
```mermaid
flowchart LR
    subgraph Internet["Public Internet (untrusted)"]
        User["Browser / User"]
    end

    subgraph DockerHost["Docker Host"]
        subgraph WebNet["Internal Docker Network (trusted)"]
            Web["Web App Container<br/>Node.js/Express<br/>Port 4000"]
            DB["MongoDB Container<br/>Port 27017"]
        end
    end

    User -- "HTTP requests<br/>(login, allocations, profile)" --> Web
    Web -- "Queries / writes<br/>(internal only, not exposed)" --> DB
```

## Trust Boundaries
1. **Public Internet ↔ Web App** — the only boundary exposed externally, on port 4000. All user input crosses here (login forms, allocation inputs) — this is the primary attack surface.
2. **Web App ↔ MongoDB** — internal Docker network only. MongoDB is never exposed to the host network directly; only the web container can reach it.

## Tech Stack
- Node.js / Express (web app)
- MongoDB (database)
- Docker & Docker Compose (containerisation)
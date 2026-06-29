dcvix is a three-component session broker and server-pool manager for Amazon DCV remote desktops. This page explains the system architecture, component relationships, and key lifecycle flows.

## Three-Tier Model

```mermaid
flowchart LR
    L[Launcher] -- HTTPS / PASETO --> D[Director]
    D -- mTLS / CA-signed certs --> A1[Agent]
    D -- mTLS / CA-signed certs --> A2[Agent]
    A1 -- DCV sessions --> W1[Workstation]
    A2 -- DCV sessions --> W2[Workstation]
    subgraph Users
        L
    end
    subgraph Central
        D
    end
    subgraph Workstations
        A1
        A2
    end
```

| Component | Role | Tech |
|-----------|------|------|
| **Director** | Central server - API, auth, session management, CA authority | Go + React frontend |
| **Agent** | Per-workstation - DCV session lifecycle, stats reporting, auto-cert management | Go |
| **Launcher** | End-user GUI - login, server list, launch DCV viewer | Go + Fyne toolkit |

## Session connection flow

```mermaid
flowchart TB
    User["User"]

    subgraph Client["Client Machine"]
        direction TB
        Launcher["dcvix Launcher"]
        Viewer["DCV Viewer"]
    end

    subgraph Control["dcvix Control Plane"]
        Director["dcvix Director"]
    end

    subgraph DCVHost["DCV Host"]
        direction TB
        Agent["dcvix Agent"]
        DCV["DCV Server<br/>(User Session)"]
    end

    User --> Launcher

    Launcher <-->|"1-3. Authenticate,<br/>list hosts, select +<br/>request session"| Director
    Director -->|"4. Create session"| Agent
    Agent -->|"5. Create"| DCV
    DCV -->|"6. Ready"| Agent
    Agent -->|"7. Session info"| Director
    Director -->|"8. Token + endpoint"| Launcher
    Launcher -->|"9. Launch with token"| Viewer
    Viewer -->|"10. Connect + present token"| DCV
    DCV <-->|"11-12. Validate token"| Director
    DCV -->|"13. Desktop session"| Viewer
```

The dcvix Launcher is the entry point for users. When a user requests a desktop session, the Launcher authenticates with the dcvix Director, which is responsible for authorization and session allocation.

The Director returns the list of available workstations and pools to the Launcher. The user selects the desired workstation. Then the Launcher asks the Director to create the session. The Director instructs the dcvix Agent running on the selected host to create a new DCV session. Once the session is ready, the Director generates a connection token and returns the required connection information to the Launcher.

The Launcher starts the DCV Viewer and passes the connection token to it. The DCV Viewer then connects directly to the DCV Server hosting the session.

During the connection process, the DCV Server validates the token by contacting the dcvix Director. The Director confirms whether the token is valid and authorized, allowing or rejecting the connection.

This architecture keeps the desktop traffic between the user and the DCV Server, while dcvix only manages authentication, authorization, and session lifecycle operations.

## User Authentication Flow

```mermaid
sequenceDiagram
    participant User
    participant Launcher
    participant Director
    participant AuthBackend as Auth Backend (PAM/LDAP/...)

    User->>Launcher: Enter credentials (+ optional OTP)
    Launcher->>Director: POST /v1/user/login
    Director->>AuthBackend: Authenticate (PAM/LDAP/RADIUS/external)
    AuthBackend-->>Director: success
    Director->>Director: Generate PASETO v4 token (2h expiry)
    Director-->>Launcher: 200 OK + PASETO token
    Launcher->>Launcher: Store token
    Launcher->>Director: GET /v1/user/servers (session cookie)
    Director->>Director: Verify token, look up policy (users.json)
    Director-->>Launcher: 200 OK (server list)
    Launcher->>User: Display available servers
```

## Session Lifecycle

```mermaid
sequenceDiagram
    participant User
    participant Launcher
    participant Director
    participant Agent
    participant DCV as DCV Server

    User->>Launcher: Select server + session type
    Launcher->>Director: Forward request (authenticated)
    Director->>Agent: POST /v1/sessions (mTLS)
    Agent->>DCV: dcv create-session <params>
    DCV-->>Agent: Session created
    Agent-->>Director: 200 OK (session info)
    Director-->>Launcher: 200 OK (session token)
    Launcher->>User: Launch dcvviewer

    Note over Agent: Periodic updates
    Agent->>Director: POST /v1/agent/update (sessions + stats)
    Note over Director: Updates in-memory session state

    User->>Launcher: Close session
    Launcher->>Director: DELETE /v1/sessions/{id}
    Director->>Agent: DELETE /v1/sessions/{id} (mTLS)
    Agent->>DCV: dcv close-session <id>
    DCV-->>Agent: Session closed
    Agent-->>Director: 200 OK
    Director-->>Launcher: 200 OK
```

## Agent Auto-Registration Flow

On first startup, an agent has no certificate. It generates a key pair and GUID locally, then registers with the director via a retry loop. The private key never leaves the agent.

```mermaid
sequenceDiagram
    participant Agent
    participant Director
    participant Admin

    Note over Agent: First startup
    Agent->>Agent: Generate Ed25519 key pair
    Agent->>Agent: Generate UUIDv4 GUID (persisted)
    Agent->>Director: POST /v1/agent/register (CSR + GUID + hostname)
    Note over Agent,Director: Plain HTTPS (TOFU or pre-deployed CA)
    Director->>Director: Verify CSR CN matches GUID
    Director->>Director: Create pending entry in agents.db
    Director-->>Agent: 403 "pending approval"
    Note over Agent: Retry every 30 seconds

    loop Every 30s
        Agent->>Director: POST /v1/agent/register (same GUID + new CSR)
        Director-->>Agent: 403 "pending approval"
    end

    Admin->>Director: Approve agent via web UI
    Note over Director: Sign CSR -> 14-day cert
    Director-->>Agent: 200 OK (signed cert + CA cert)
    Agent->>Agent: Store agent.crt + ca.pem
    Note over Agent: Switch to strict mTLS
    Agent->>Agent: Start mTLS server + renewal loop
```

Flow details:

- **Key generation**: Ed25519, stored at `{dataDir}/agent.key` (0600)
- **GUID**: UUIDv4, stored at `{dataDir}/agent.guid`, survives restarts
- **CSR**: CN = `dcvix-agent-{guid}`, DER-encoded, base64 in JSON
- **Director validation**: Parses CSR, verifies CN matches submitted GUID (binds GUID to public key)
- **Agent states**: `pending` -> `registered` -> `revoked`

## Certificate Renewal Flow

Once registered, the agent renews its certificate periodically (default every 12 hours). The renewal uses the existing mTLS connection.

```mermaid
sequenceDiagram
    participant Agent
    participant Director

    Note over Agent: Every ~12 hours
    Agent->>Agent: Check CertificateNeedsRenewal() (<24h to expiry)
    Agent->>Agent: Generate new CSR (same key pair)
    Agent->>Director: POST /v1/agent/renew (CSR)
    Note over Agent,Director: mTLS (current cert)
    Director->>Director: Verify mTLS cert CN = GUID
    Director->>Director: Verify agent state = registered
    Director->>Director: Verify CSR CN matches mTLS GUID
    Director->>Director: Sign new CSR -> 14-day cert
    Director-->>Agent: 200 OK (signed cert)
    Agent->>Agent: Store new agent.crt
```

The mTLS server uses a `GetCertificate` callback so the renewed cert is picked up without restarting the listener.

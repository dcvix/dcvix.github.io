This section covers everything you need to install, configure, and run your first dcvix deployment.

## Prerequisites

- One Linux server for the **director** (the central management service)
- One or more Linux or Windows workstations with Amazon DCV installed for **agents**
- End-user desktops (Linux, Windows, or macOS) for the **launcher**

## Installation flow

```mermaid
flowchart TD

subgraph Setup
A[Install Director] --> B[Start Director]
C[Install Agent] --> D[Start Agent]
end

subgraph Enrollment
D --> E[Generate Key + GUID]
E --> F[Approve in Director UI]
end

subgraph User Flow
G[Install Launcher] --> H[Configure Director URL]
H --> I[Login]
I --> J[Select DCV Server]
J --> K[Connect Session]
end

B --> F
F --> G
```

- [Quickstart](quickstart.md) - get running in 5 minutes
- [Installation](installation.md) - detailed install options per platform

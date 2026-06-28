> [!WARNING]
> dcvix is under active development. Do not deploy in production environments unless you are prepared to handle incomplete features, breaking changes, and unexpected behavior.


![Logo](assets/images/dcvixLogoDarkBG.png)

dcvix is a session and server-pool manager for Amazon DCV. It provides centralized authentication, desktop session lifecycle management, and automatic allocation of DCV servers.
It consists of three components:

- The **director** runs on a central server and allows administrators to control desktop access and user authentication. It also acts as a token authenticator.

- The **agent** runs on workstations, collecting statistics and session information to be sent to the director. It responds to requests for session creation and termination.

- The **launcher** runs on the user's computer with a GUI. It authenticates users to obtain a security token, displays a list of DCV servers available to the user, and launches the DCV viewer.

## Quick Links

- [Why dcvix exists?](getting-started/why.md) - Brief introduction
- [Getting Started](getting-started/quickstart.md) - run dcvix in 5 minutes
- [Architecture Overview](architecture/overview.md) - understand the system
- [Components](components/director.md) - what each piece does
- [Configuration](configuration/index.md) - how to control it


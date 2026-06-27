> [!WARNING]
> This project is a work in progress. It is advised that you do not use this in production
> unless you are willing to encounter bugs, papercuts or your computer on fire along the way.


![Logo](assets/images/DcvixLogoDarkBG.png)

dcvix is an orchestrator for the Amazon DCV remote desktop system focused on simplicity, lightweight operation, and security.
It enables small to medium-sized organizations to manage the creation of sessions and server pools.
It consists of three components:

- The **director** runs on a central server and allows administrators to control desktop access and user authentication. It also acts as a token authenticator.

- The **agent** runs on workstations, collecting statistics and session information to be sent to the director. It responds to requests for session creation and termination.

- The **launcher** runs on the user's computer with a GUI. It authenticates users to obtain a security token, displays a list of DCV servers available to the user, and launches the DCV viewer.

## Quick Links

- [Getting Started](getting-started/quickstart.md) - run dcvix in 5 minutes
- [Architecture Overview](architecture/overview.md) - understand the system
- [Components](components/director.md) - what each piece does
- [Configuration](configuration/index.md) - how to control it


> [!WARNING]
> This project is a work in progress. It is advised that you do not use this in production
> unless you are willing to encounter bugs, papercuts or your computer on fire along the way.

<p align="center">
  <img src="docs/assets/images/dcvixLogoDarkBG.png" width="300" alt="Logo">
</p>

dcvix is an orchestrator for the Amazon DCV remote desktop system focused on simplicity, lightweight operation, and security.
It enables small to medium-sized organizations to manage the creation of sessions and server pools.
It consists of three components:
- The **director** runs on a central server and allows administrators to control desktop access and user authentication. It also acts as a token authenticator.
- The **agent** runs on workstations, collecting statistics and session information to be sent to the director. It responds to requests for session creation and termination.
- The **launcher** runs on the user's computer with a GUI. It authenticates users to obtain a security token, displays a list of DCV servers available to the user, and launches the DCV viewer.

dcvix Docs
===========

This repo contains all the documentation for the dcvix project built with MkDocs. Visit the [website](https://dcvix.github.io/) to view the documentation.

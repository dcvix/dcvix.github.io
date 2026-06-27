## NSIS Installer

The agent is distributed as an NSIS installer (`dcvix-agent-<version>-setup.exe`). Run the installer to:

- Install the binary to `C:\Program Files\dcvix\Agent\`
- Create a Windows Service for automatic startup
- Generate a default config at `%ProgramData%\dcvix\Agent\dcvix-agent.conf`
- Configure logging at `%ProgramData%\dcvix\Agent\log`
- Add an entry in Add/Remove Programs for uninstallation

## Service Management

```cmd
net start dcvix-agent
net stop dcvix-agent
```

Or manage via the Services console (`services.msc`): look for **dcvix Agent**.

## File Paths

| Component | Config | Data | Logs |
|-----------|--------|------|------|
| Director | `%ProgramData%\dcvix\Director\dcvix-director.conf` | `%ProgramData%\dcvix\Director\` | `%ProgramData%\dcvix\Director\log\` |
| Agent | `%ProgramData%\dcvix\Agent\dcvix-agent.conf` | `%ProgramData%\dcvix\Agent\` | `%ProgramData%\dcvix\Agent\log\` |

`%ProgramData%` typically resolves to `C:\ProgramData`.

## Networking

| Port | Component | Direction | Purpose |
|------|-----------|-----------|---------|
| `8445` | Director | Inbound | HTTPS API + admin UI |
| `8446` | Agent | Inbound | mTLS API for director session operations |

Windows Firewall rules should be created for both ports as needed.

## Launcher

The launcher is a GUI application and does not run as a service. Run the launcher executable from the installation directory.

Configuration file `dcvix-launcher.conf` is discovered in the user config directory (`%AppData%/net.cortassa.dcvix-launcher/`), the working directory, or the executable directory, in that order.

## Building the Installer

Requires NSIS installed:

```bash
# On Ubuntu/Debian
sudo apt install nsis

# On Fedora
sudo dnf install mingw32-nsis
```

```bash
cd dcvix-agent
make installer
```

The installer is created at `bin/installer/dcvix-agent-<version>-setup.exe`.

## Cross-Build from Linux

```bash
cd dcvix-launcher
make build-windows-cross
```

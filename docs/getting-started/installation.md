## Linux Packages

### Debian / Ubuntu

```bash
sudo dpkg -i dcvix-director_<version>.deb
sudo dpkg -i dcvix-agent_<version>.deb
```

### Rocky Linux / RHEL

```bash
sudo rpm -i dcvix-director-<version>.rpm
sudo rpm -i dcvix-agent-<version>.rpm
```

### Installed Paths (Linux)

| Component | Binary | Config | Data | Logs |
|-----------|--------|--------|------|------|
| Director | `/usr/bin/dcvix-director` | `/etc/dcvix-director/dcvix-director.conf` | `/var/lib/dcvix-director/` | `/var/log/dcvix-director/` |
| Agent | `/usr/bin/dcvix-agent` | `/etc/dcvix-agent/dcvix-agent.conf` | `/var/lib/dcvix-agent/` | `/var/log/dcvix-agent/` |

Services are registered with systemd automatically:

```bash
sudo systemctl enable --now dcvix-director
sudo systemctl enable --now dcvix-agent
```

## Windows

### NSIS Installer

Run the installer `dcvix-agent-<version>-setup.exe`. It:

- Installs the binary to `C:\Program Files\dcvix\Agent\`
- Creates a Windows Service for automatic startup
- Creates the config at `%ProgramData%\dcvix\Agent\dcvix-agent.conf`
- Configures logging at `%ProgramData%\dcvix\Agent\log`
- Adds an entry in Add/Remove Programs

```cmd
net start dcvix-agent
```

### Installed Paths (Windows)

| Component | Binary | Config | Data | Logs |
|-----------|--------|--------|------|------|
| Director | `C:\Program Files\dcvix\Director\` | `%ProgramData%\dcvix\Director\dcvix-director.conf` | `%ProgramData%\dcvix\Director\` | `%ProgramData%\dcvix\Director\log\` |
| Agent | `C:\Program Files\dcvix\Agent\` | `%ProgramData%\dcvix\Agent\dcvix-agent.conf` | `%ProgramData%\dcvix\Agent\` | `%ProgramData%\dcvix\Agent\log\` |

## macOS (Launcher Only)

The launcher is distributed as a tarball:

```bash
tar -xzf dcvix-launcher-v<version>-darwin-amd64.tar.gz
cd dcvix-launcher
```

## Build from Source

### Build Requirements

| Component | Requirements |
|-----------|-------------|
| Director | Go 1.22+, Node.js 22+, npm 10+, SQLite3, libpam-dev |
| Agent | Go 1.23+ |
| Launcher | Go 1.26+, gcc, libgl1-mesa-dev, xorg-dev |

### Build Commands

```bash
# Director
cd dcvix-director && make build

# Agent
cd dcvix-agent && make build-linux

# Launcher
cd dcvix-launcher && make build-linux
```

### Cross-Build Launcher for Windows

```bash
go install github.com/fyne-io/fyne-cross@latest
cd dcvix-launcher && make build-windows-cross
```

### Build Packages

```bash
# Debian packages
make deb

# RPM packages
make rpm

# Windows NSIS installer
make installer
```

See each component's `README.md` for platform-specific build instructions using docker or podmand for reproducible builds.

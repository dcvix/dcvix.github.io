## Requirements

- Go 1.23 or higher

## Build

```bash
make build
```

Before building, a custom default configuration can be embedded in the binary by editing `internal/config/dcvix-agent.conf.default`. This lets you ship agents with pre-configured `director_host`, tags, or other settings out of the box.

## Run

```bash
make run
```

## Make Targets

| Target | Description |
|--------|-------------|
| `build` | Build binary (platform-agnostic) |
| `run` | Build and run |
| `clean` | Remove build artifacts |
| `installer` | Build Windows NSIS installer |

## Building Packages

### RockyLinux RPM

Using docker or podman:

```bash
podman build -t go-rpm-build -f contrib/rpm/Dockerfile .
podman run -it --rm -v "$PWD":/workspace -w /workspace go-rpm-build bash
make rpm
```

### Ubuntu DEB

Using docker or podman:

```bash
podman run -it --rm -v "$PWD":/workspace ubuntu:24.04 bash
apt update
apt install -y ca-certificates make curl
curl -L -O https://go.dev/dl/go1.26.3.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.26.3.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
cd /workspace
make deb
```

### Windows Installer

Requires NSIS (Nullsoft Scriptable Install System):

```bash
# On Ubuntu/Debian
sudo apt install nsis
# On Fedora
sudo dnf install mingw32-nsis
```

```bash
make installer
```

The installer is created in `bin/installer/dcvix-agent-<version>-setup.exe`.

The Windows installer:
- Installs the agent to `C:\Program Files\dcvix\Agent` by default
- Creates a Windows service for automatic startup
- Configures logging directory in `%ProgramData%\dcvix\Agent\log`
- Adds an entry to Add/Remove Programs for easy uninstallation
- Uses default configuration at `%ProgramData%\dcvix\Agent\dcvix-agent.conf`

## Releases

The project uses GitHub Actions to automatically build and publish releases. When a new version is ready:

1. Update CHANGELOG.md
2. Commit all changes
3. Create and push a new tag:

```bash
git tag v<version>
git push origin v<version>
```

The version is derived from the git tag (the `v` prefix is stripped).

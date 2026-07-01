## Requirements

- Go 1.23 or higher

## Build

```bash
make build
```

Before building, a custom default configuration can be embedded in the binary by editing `internal/config/dcvix-agent.conf.default`. This lets you ship agents with pre-configured `director_host`, tags, or other settings out of the box.

## Make Targets

| Target | Description |
|--------|-------------|
| `build` | Build binary (all supported platforms) |
| `clean` | Remove build artifacts |
| `installer` | Build Windows NSIS installer |
| `deb` | Build Debian package |
| `rpm` | Build RPM package |

## Building Packages

### Rocky Linux / RHEL RPM

Using docker or podman:

```bash
podman build -t go-rpm-build -f contrib/rpm/Dockerfile .
podman run -it --rm -v "$PWD":/workspace -w /workspace go-rpm-build bash
make rpm
```

```bash
podman run -it --rm -v "$PWD":/workspace -w /workspace rockylinux:9
dnf install -y bash rpm-build systemd systemd-rpm-macros rpmdevtools make gcc which git golang
curl -L https://go.dev/dl/go1.26.3.linux-amd64.tar.gz | tar -zx -C /usr/local
export PATH=$PATH:/usr/local/go/bin
make rpm
```

### Ubuntu DEB

Using docker or podman:

```bash
podman run -it --rm -v "$PWD":/workspace -w /workspace ubuntu:24.04 bash
apt update && apt install -y ca-certificates make curl git
curl -L https://go.dev/dl/go1.26.3.linux-amd64.tar.gz | tar -zx -C /usr/local
export PATH=$PATH:/usr/local/go/bin
make deb
```

### Windows Installer

Requires NSIS (Nullsoft Scriptable Install System):

Using docker or podman:

```bash
podman run -it --rm -v "$PWD":/workspace -w /workspace ubuntu:24.04 bash
apt update && apt install -y ca-certificates make curl git nsis p7zip-full 
curl -L https://go.dev/dl/go1.26.3.linux-amd64.tar.gz | tar -zx -C /usr/local
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
go install github.com/tc-hib/go-winres@latest
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
git tag v<VERSION>
git push origin v<VERSION>
```

The version is derived from the git tag (the `v` prefix is stripped).

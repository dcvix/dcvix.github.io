## Requirements

- Go 1.26 or higher
- gcc
- libgl1-mesa-dev
- xorg-dev

```bash
sudo apt-get install gcc libgl1-mesa-dev xorg-dev
```

## Build

```bash
go install fyne.io/tools/cmd/fyne@latest
go mod tidy
make
```

Before building, a custom default configuration can be embedded in the binary by editing `internal/config/dcvix-launcher.conf.default`. This lets you ship the launcher with pre-configured broker URL, viewer command, or other settings out of the box.

## Run

```bash
make run
```

## Make Targets

| Target | Description |
|--------|-------------|
| `build-linux` | Build Linux binary |
| `run` | Build and run |
| `clean` | Remove build artifacts |
| `installer` | Build Windows NSIS installer |
| `deb` | Build Debian package |
| `rpm` | Build RPM package |

## Cross Build for Windows

Requires [fyne-cross](https://github.com/fyne-io/fyne-cross):

```bash
go install github.com/fyne-io/fyne-cross@latest
make build-windows-cross
```

## Building Packages

### RockyLinux RPM

Using docker or podman:

```bash
podman run -it --rm -v "$PWD":/workspace rockylinux:9 bash
dnf install -y 'dnf-command(config-manager)'
dnf config-manager --set-enabled crb
dnf install -y \
    golang \
    gcc \
    rpm-build \
    make \
    which \
    desktop-file-utils \
    libX11-devel \
    libxkbcommon-devel \
    mesa-libGL-devel \
    wayland-devel \
    libXcursor-devel \
    libXi-devel \
    libXinerama-devel \
    libXrandr-devel \
    libXxf86vm-devel
curl -L https://go.dev/dl/go1.26.3.linux-amd64.tar.gz | tar -zx -C /usr/local
export PATH=/usr/local/go/bin:$PATH
cd /workspace
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

Requires NSIS:

```bash
sudo apt install nsis
make installer
```

The installer is created in `bin/installer/dcvix-launcher-<version>-setup.exe`.

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

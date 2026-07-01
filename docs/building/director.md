## Requirements

- Go 1.22 or higher
- Node.js 22 or higher
- npm 10 or higher
- SQLite3
- libpam-dev
  - Debian/Ubuntu: `sudo apt-get install libpam-dev`
  - RedHat: `sudo dnf install pam-devel`

## Build

```bash
make build
```

This builds the frontend and creates a Linux executable in the `dist/` directory.

## Frontend

The frontend is a single-page application built with React. It is served by the director and provides a web interface for administrators to manage sessions and servers.

To build the frontend separately:

```bash
make frontend
```

## Make Targets

| Target | Description |
|--------|-------------|
| `build` | Build frontend + Go binary |
| `frontend` | Build only the React frontend |
| `audit` | Run quality checks |
| `clean` | Remove build artifacts |
| `deb` | Build Debian package |
| `rpm` | Build RPM package |

## Building Packages

### Rocky Linux / RHEL RPM

Using docker or podman:

```bash
podman run -it --rm -v "$PWD":/workspace -w /workspace rockylinux:9 bash
dnf install -y bash rpm-build systemd systemd-rpm-macros rpmdevtools make gcc which git golang pam-devel npm
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

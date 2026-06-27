This section covers setting up a build environment for dcvix components from source.

## Build Requirements by Component

| Component | Go | Other requirements |
|-----------|----|-------------------|
| Director | 1.22+ | Node.js 22+, npm 10+, SQLite3, libpam-dev |
| Agent | 1.23+ | None |
| Launcher | 1.26+ | gcc, libgl1-mesa-dev, xorg-dev (Linux), or fyne-cross (cross-build) |

## Installing Go

### Linux

```bash
# Download the latest Go (check https://go.dev/dl/ for current version)
wget https://go.dev/dl/go1.26.3.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.26.3.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
```

### Windows

Download the MSI installer from https://go.dev/dl/ and run it. Go is installed to `C:\Program Files\Go` by default, and the installer adds it to your PATH.

## Installing System Dependencies

### Linux (Debian / Ubuntu)

```bash
sudo apt-get update
sudo apt-get install -y \
    gcc \
    make \
    libgl1-mesa-dev \
    xorg-dev \
    libpam-dev \
    npm
```

### Linux (Rocky Linux / RHEL)

```bash
sudo dnf install -y \
    gcc \
    make \
    mesa-libGL-devel \
    libxkbcommon-devel \
    wayland-devel \
    libXcursor-devel \
    libXi-devel \
    libXinerama-devel \
    libXrandr-devel \
    libXxf86vm-devel \
    pam-devel \
    npm
```

### Windows

Install **gcc** via [MinGW-w64](https://www.mingw-w64.org/) or [TDM-GCC](https://jmeubank.github.io/tdm-gcc/). The NSIS installer builder requires [NSIS](https://nsis.sourceforge.io/).

## Verifying the Setup

```bash
go version
make --version
gcc --version
node --version
npm --version
```

## Next Steps

See [Building Director](director.md), [Building Agent](agent.md), or [Building Launcher](launcher.md) for per-component build instructions.

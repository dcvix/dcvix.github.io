The launcher configuration file (`dcvix-launcher.conf`) uses INI format with the following sections.

## `[dcvix-launcher]`

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `broker` | string | *(required)* | Director URL. **Must use `https://`** (e.g. `https://director.lan:8445`) |
| `user_id` | string | `""` | Pre-filled username in the login form |
| `otp` | bool | `False` | Whether a one-time password is required at login |
| `command` | string | `dcvviewer` | Path to the DCV viewer executable |
| `accept-untrusted-cert` | bool | `false` | Allow self-signed director certificates (not recommended for production) |
| `allow-custom-server` | bool | `false` | Let users type an arbitrary server hostname |

## `[log]`

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `level` | string | `info` | Log verbosity: `debug`, `info`, `warn`, `error`, `fatal` |

## Example Config

```ini
[dcvix-launcher]
broker=https://director.example.lan:8445
user_id=jdoe
otp=False
command="dcvviewer"
accept-untrusted-cert=false
allow-custom-server=false
```

The launcher looks for `dcvix-launcher.conf` in this order:

1. Path passed via `--config` flag
2. OS-specific user config directory:
   - Linux: `~/.config/net.cortassa.dcvix-launcher/dcvix-launcher.conf`
   - macOS: `~/Library/Application Support/net.cortassa.dcvix-launcher/dcvix-launcher.conf`
   - Windows: `%AppData%/net.cortassa.dcvix-launcher/dcvix-launcher.conf`
3. Current working directory
4. Executable directory

If no file is found, a default config with commented-out options is written to the user config directory (item 2) automatically.

## Related

- [Launcher component](../components/launcher.md) - internal architecture
- [Configuration overview](index.md) - discovery and conventions

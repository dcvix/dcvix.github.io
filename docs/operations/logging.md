## Log Levels

All components support configurable log verbosity:

| Level | Usage |
|-------|-------|
| `debug` | Detailed diagnostic information |
| `info` | Normal operational messages |
| `warning` | Recoverable issues that warrant attention |
| `error` | Non-fatal failures |
| `critical` | Fatal errors requiring immediate action |

## Log Configuration

Each component's config file has a `[log]` section:

```ini
[log]
level = info
directory = /var/log/dcvix-director
rotation = 2
```

- **`level`** - controls verbosity (default: `info`)
- **`directory`** - output path (default varies by platform and component)
- **`rotation`** - number of 100 MB log files to retain (default: `2`)

## Log Locations

### Director

| Platform | Default path |
|----------|-------------|
| Linux | `/var/log/dcvix-director/` |
| Windows | `%ProgramData%\dcvix\Director\log\` |

### Agent

| Platform | Default path |
|----------|-------------|
| Linux | `/var/log/dcvix-agent/` |
| Windows | `%ProgramData%\dcvix\Agent\log\` |

### Launcher

| Platform | Default path |
|----------|-------------|
| Linux | Working directory |
| Windows | Working directory |
| macOS | Working directory |

## Key Log Entries to Watch

### Director

| Event | Log pattern | Level |
|-------|-------------|-------|
| CA auto-generated | `"CA not found, generating new CA"` | info |
| CA loaded | `"CA loaded from data_dir"` | info |
| Server start | `"Server listening on :8445"` | info |
| Agent registration | `"Registration: created pending agent <guid>"` | info |
| Agent re-registration | `"Registration: agent <guid> is already registered, re-signing"` | info |
| Token key missing | `"No token_key found in config!"` | warning |
| SIGHUP | `"Reloading policy database"` | info |
| Agent approval | (via admin UI, logged as info) | info |

### Agent

| Event | Log pattern | Level |
|-------|-------------|-------|
| Key generation | `"Generated new Ed25519 key pair"` | info |
| Registration pending | `"Registration: pending approval (GUID: ...)"` | info |
| Registration success | `"Registration: agent <guid> registered successfully"` | info |
| Registration failure | `"Registration: connection failed"` | warning |
| Renewal | `"Certificate renewed, next renewal in 12h"` | info |
| Renewal failure | `"Renewal failed, will retry in 5 minutes"` | warning |
| Cert expiry warning | `"Certificate expires in < 24h, renewing"` | info |
| Stat update | `"Sending update to director"` | debug |

## Viewing Logs

### Linux (systemd)

```bash
# Follow logs
journalctl -u dcvix-director -f
journalctl -u dcvix-agent -f

# Last 50 lines
journalctl -u dcvix-director -n 50 --no-pager

# Filter by time
journalctl -u dcvix-director --since "5 minutes ago"
```

### File-based logs

If log files are enabled, they are written to the configured `directory`:

```bash
tail -f /var/log/dcvix-director/dcvix-director.log
tail -f /var/log/dcvix-agent/dcvix-agent.log
```

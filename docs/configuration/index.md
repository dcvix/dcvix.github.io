All dcvix components use INI-format configuration files. Each component has its own config file with component-specific sections.

## Config Discovery

The configuration file is discovered in this order:

1. Path specified by the `--conf` command-line flag
2. Platform-specific default path:
   - Linux: `/etc/dcvix-{component}/dcvix-{component}.conf`
   - Windows: `%ProgramData%\dcvix\{Component}\dcvix-{component}.conf`
3. `dcvix-{component}.conf` in the working directory

If no config file exists, a default one is generated with commented-out options.

## Policy Database (Director Only)

The director reads user and pool policies from JSON files:

```text
/etc/dcvix-director/policydb/
  users.json   # User-to-server and user-to-pool assignments
  pools.json   # Pool definitions (group of servers)
```

These files are loaded into an in-memory SQLite database at startup. Send `SIGHUP` to the director process to reload them without restarting.

## Data Directory

All three components use a `data_dir` setting for persistent state. Default paths:

| Component | Linux | Windows |
|-----------|-------|---------|
| Director | `/var/lib/dcvix-director` | `%ProgramData%\dcvix\Director` |
| Agent | `/var/lib/dcvix-agent` | `%ProgramData%\dcvix\Agent` |
| Launcher | `~/.config/net.cortassa.dcvix-launcher` | `%AppData%/net.cortassa.dcvix-launcher` |

The `data_dir` is created automatically at startup if it does not exist.

## Removed Fields

The following certificate-related config fields were removed in the auto-registration model:

- `ca_file` - CA cert is auto-managed in `data_dir` (`ca.pem`)
- `ssl_cert_file`, `ssl_key_file` - server/agent certs are auto-managed in `data_dir`
- `tls_strict` - strictness is derived from operational state (TOFU vs registered)

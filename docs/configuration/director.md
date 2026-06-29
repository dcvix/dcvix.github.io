The director configuration file (`dcvix-director.conf`) uses INI format with the following sections.

## `[director]`

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `director_host` | string | `0.0.0.0` | Hostname or IP the director listens on |
| `director_port` | int | `8445` | HTTPS listener port |
| `agent_port` | int | `8446` | Port agents listen on (used for outbound connections) |
| `auth_type` | string | `pam` | Authentication backend: `pam`, `ldap`, `radius`, `external` |
| `policydb_folder` | string | `policydb` | Path to the policy JSON directory |
| `data_dir` | string | `/var/lib/dcvix-director` | Persistent state directory (CA keys, agents.db, server cert) |
| `token_key` | string | *(auto-generated)* | Base64-encoded PASETO symmetric key. **Set this to persist tokens across restarts.** |

## `[pam_auth]`

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `pam_service_name` | string | `login` | PAM service name |

## `[ldap_auth]`

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `ldap_address` | string | `ldap://127.0.0.1:389` | LDAP server URL |
| `ldap_base_dn` | string | `dc=example,dc=com` | LDAP base DN |
| `ldap_bind_user` | string | `cn=admin,dc=example,dc=com` | Bind DN |
| `ldap_bind_password` | string | `password` | Bind password |
| `ldap_filter` | string | `(sAMAccountName=%s)` | Search filter (`%s` = userID) |
| `otp_type` | string | `disabled` | OTP backend: `disabled`, `privacyidea`, `external` |
| `privacyidea_url` | string | `https://mfaserver.domain.com` | PrivacyIDEA server URL (calls `POST /validate/check`) |
| `privacyidea_tls_strict` | bool | `true` | Verify PrivacyIDEA TLS certificate |
| `otp_external_command` | string | `/usr/bin/external-auth` | External OTP verification command |
| `otp_external_args` | string | `""` | Arguments for external OTP command |

## `[radius_auth]`

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `radius_server` | string | `127.0.0.1` | RADIUS server address |
| `radius_port` | int | `1812` | RADIUS server port |
| `radius_secret` | string | `secret` | RADIUS shared secret |

## `[external_auth]`

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `external_command` | string | `/usr/bin/external-auth` | External auth binary path |
| `args` | string | `""` | Command-line arguments |

The external command receives `UserID\nPassword\nOTP\n` on stdin and must exit 0 for success, 1 for failure.

## `[gateway]`

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `gateways_list` | string | `127.0.0.1` | Comma-separated IPs allowed to access `/resolveSession` |

## `[housekeeper]`

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `housekeeper_frequency` | duration | `40s` | Housekeeper run interval |
| `max_age` | duration | `30s` | Max age of stale sessions before cleanup |

## `[log]`

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `level` | string | `info` | Log verbosity: `debug`, `info`, `warning`, `error`, `critical` |
| `directory` | string | `log` | Log output directory |
| `rotation` | int | `2` | Number of 100 MB log files to keep |

## Policy JSON Files

### `policydb/users.json`

```json
[
    {
        "UserID": "user1",
        "Workstations": ["ws1.domain.com", "ws2.domain.com"],
        "Pools": ["testing"]
    },
    {
        "UserID": "user2",
        "Workstations": ["ws3.domain.com"],
        "Pools": ["engineering"]
    }
]
```

Special workstation `ALLOW_CUSTOM` grants access to all servers (launcher shows a custom hostname input).

### `policydb/pools.json`

```json
[
    {
        "PoolID": "testing",
        "Workstations": ["ws5.domain.com", "ws6.domain.com"]
    },
    {
        "PoolID": "engineering",
        "Workstations": ["ws8.domain.com", "ws9.domain.com"]
    }
]
```

## Example Minimal Config

```ini
[director]
director_host = "0.0.0.0"
director_port = 8445
auth_type = "pam"
policydb_folder = "/etc/dcvix-director/policydb"
data_dir = "/var/lib/dcvix-director"
```

## Related

- [Director component](../components/director.md) - internal architecture
- [Configuration overview](index.md) - discovery and conventions

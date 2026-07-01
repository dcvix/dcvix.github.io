## Package Installation

Install via your distribution's package manager:

```bash
# Debian / Ubuntu
sudo dpkg -i dcvix-director_<version>.deb
sudo dpkg -i dcvix-agent_<version>.deb

# Rocky Linux / RHEL
sudo rpm -i dcvix-director-<version>.rpm
sudo rpm -i dcvix-agent-<version>.rpm
```

## Service Management

Both components register as systemd services (package install or `make install`):

```bash
sudo systemctl enable --now dcvix-director
sudo systemctl enable --now dcvix-agent
```

Common operations:

```bash
sudo systemctl status dcvix-director
sudo systemctl restart dcvix-agent
sudo journalctl -u dcvix-director -f
```

### Run Without Installing

```bash
# Director
dcvix-director --conf path/to/dcvix-director.conf

# Agent
dcvix-agent --conf path/to/dcvix-agent.conf
```

## File Paths

| Component | Config | Data | Logs |
|-----------|--------|------|------|
| Director | `/etc/dcvix-director/dcvix-director.conf` | `/var/lib/dcvix-director/` | `/var/log/dcvix-director/` |
| Agent | `/etc/dcvix-agent/dcvix-agent.conf` | `/var/lib/dcvix-agent/` | `/var/log/dcvix-agent/` |

The policy directory for the director:

```text
/etc/dcvix-director/policydb/
  users.json
  pools.json
```

## Networking

| Port | Component | Direction | Purpose |
|------|-----------|-----------|---------|
| `8445` | Director | Inbound | HTTPS API + admin UI. Accepts connections from agents, launchers, and admin browsers. |
| `8446` | Agent | Inbound | mTLS API for director session operations. Accepts connections from the director only. |

### Firewall

```bash
# Director
sudo firewall-cmd --add-port=8445/tcp --permanent
sudo firewall-cmd --reload

# Agent (if on a different subnet from the director)
sudo firewall-cmd --add-port=8446/tcp --permanent
sudo firewall-cmd --reload
```

When deploying agents and director on the same internal network, port `8446` can be restricted to the director's IP:

```bash
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="<director-ip>" port port="8446" protocol="tcp" accept'
```

## Configure DCV Server for Auth

Each DCV server needs to trust the director's CA and use the director as the auth token verifier.

Extract the CA certificate from the director:

```bash
openssl s_client -connect director.mydomain.com:8445 2>/dev/null \
  | openssl x509 > /etc/dcv/dcvix-ca.pem
```

Add to `/etc/dcv.conf`:

```ini
[security]
ca-file="/etc/dcv/dcvix-ca.pem"
no-tls-strict = true
auth-token-verifier="https://director.mydomain.com:8445/v1/user/authtokenverify"
```

This enables DCV sessions to validate auth tokens issued by the director.

## Data Directory Contents

### Director: `/var/lib/dcvix-director/`

| File | Purpose | Managed by |
|------|---------|------------|
| `ca.key` | CA private key (0600) | Auto-generated on first startup |
| `ca.pem` | CA certificate | Auto-generated on first startup |
| `server.key` | Server private key (0600) | Auto-generated with CA |
| `server.crt` | Server certificate (CN/SAN from `hostname -f`) | Auto-generated with CA |
| `agents.db` | Agent registration database (SQLite) | Created on first startup |

### Agent: `/var/lib/dcvix-agent/`

| File | Purpose | Managed by |
|------|---------|------------|
| `agent.key` | Ed25519 private key (0600) | Auto-generated on first startup |
| `agent.guid` | UUIDv4 agent identity | Auto-generated on first startup |
| `agent.crt` | Signed certificate | Received from director on registration |
| `ca.pem` | CA certificate | Received from director on registration |

## Backups

For disaster recovery, back up these directories on the **director**:

- `/var/lib/dcvix-director/` - CA keys and agent database (critical)
- `/etc/dcvix-director/dcvix-director.conf` - configuration
- `/etc/dcvix-director/policydb/` - user and pool policies

Without the CA key backup, a director recovery requires all agents to re-register.
Without `token_key` in the config, all user sessions become invalid on restart.

## Building from Source

See [Installation > Build from Source](../getting-started/installation.md#build-from-source) for build requirements and commands.

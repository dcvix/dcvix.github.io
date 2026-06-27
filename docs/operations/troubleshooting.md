## Agent won't register

**Symptoms:** Agent logs show `"Registration: connection failed"` or hangs at `"pending approval"`.

**Checks:**

1. Verify the director is reachable from the agent: `curl -k https://<director>:8445/v1/health`
2. Check `director_host` in `/etc/dcvix-agent/dcvix-agent.conf` is correct
3. Ensure port `8445` is open on the director's firewall
4. Check director logs for the registration request (`journalctl -u dcvix-director -f`)
5. Visit `https://<director>:8445/admin/agents` - if the agent appears as pending, click Approve

## Agent stuck in "pending approval" loop

**Symptom:** Agent logs `"Registration: pending approval"` repeatedly over many minutes.

**Cause:** The agent's GUID is in `pending` state and no admin has approved it.

**Fix:** Log into the director admin UI and approve the agent. If the agent is not listed, check the agent's GUID in `/var/lib/dcvix-agent/agent.guid` and verify in the database:

```bash
sqlite3 /var/lib/dcvix-director/agents.db "SELECT * FROM agents;"
```

## All sessions invalid after director restart

**Symptom:** After restarting the director, all users are logged out and cannot log in, or get "token invalid" errors.

**Cause:** The `token_key` was not set in `dcvix-director.conf`. On each startup without a configured key, a new ephemeral key is generated, invalidating all existing PASETO tokens.

**Fix:** Find the generated key in the director's startup logs (look for `"token_key = ..."` in a warning block) and add it to the config:

```ini
[director]
token_key = "<base64-key>"
```

Restart the director. All users will need to log in again.

## Agent cert expired, won't reconnect

**Symptom:** Agent logs show certificate errors or mTLS handshake failures.

The agent should automatically fall back to re-registration. If it doesn't:

1. Check the agent's cert: `openssl x509 -in /var/lib/dcvix-agent/agent.crt -text -noout | grep -A2 Validity`
2. If expired, remove the cert file and restart the agent:
   ```bash
   sudo rm /var/lib/dcvix-agent/agent.crt
   sudo systemctl restart dcvix-agent
   ```
3. The agent generates a new CSR with its existing GUID and key, registers again, and requires admin approval in the UI.

## Director `agents.db` lost

**Symptom:** Director logs show agent registrations failing even though agents were previously approved.

The database is recreated empty on startup. All existing agents try to renew, the director rejects them (unknown GUID). Agents fall back to fresh registration using their persisted GUID and key.

**Fix:** Approve the re-registering agents in the admin UI.

## Director CA key lost

**Symptom:** Director logs show a new CA was generated on startup. Existing agents cannot connect (their certs were signed by the old CA).

Both `ca.key` and `ca.pem` were lost. The director auto-generates a new CA, but all existing agent certs become invalid.

**Fix:** All agents must re-register. On each agent:

```bash
sudo rm /var/lib/dcvix-agent/agent.crt /var/lib/dcvix-agent/ca.pem
sudo systemctl restart dcvix-agent
```

Then approve each agent in the admin UI.

## Key permission errors

**Symptom:** Director fails to start with `"key file has group/other permissions"`.

CA and server keys must have `0600` permissions:

```bash
sudo chmod 0600 /var/lib/dcvix-director/ca.key
sudo chmod 0600 /var/lib/dcvix-director/server.key
```

Agent keys similarly:

```bash
sudo chmod 0600 /var/lib/dcvix-agent/agent.key
```

## Firewall / connectivity issues

**Symptom:** Agents can't reach the director, or the director can't reach agents.

| Port | Direction | Purpose |
|------|-----------|---------|
| `8445` | Inbound to director | Agent registration, launcher connections, admin UI |
| `8446` | Inbound to agent | Director session management requests |

Verify with:

```bash
# From agent to director
curl -k https://<director>:8445/v1/health

# From director to agent (requires mTLS cert)
curl --cert /var/lib/dcvix-director/server.crt --key /var/lib/dcvix-director/server.key --cacert /var/lib/dcvix-director/ca.pem https://<agent>:8446/v1/health
```

## Launcher shows "no servers available"

**Symptom:** User logs in successfully but the server list is empty.

**Checks:**

1. Verify the user is assigned to workstations or pools in `policydb/users.json`
2. Ensure the assigned workstations match the agent's hostname (as reported during registration)
3. Verify the agent has registered successfully and its state is `registered` (not `pending`)
4. Check the agent is reporting stats via `POST /v1/agent/update` - the director marks agents as active only when they send updates
5. After editing policy files, send SIGHUP to the director: `sudo killall -HUP dcvix-director`

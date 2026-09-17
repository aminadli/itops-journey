## Planned Tech Stack


| Layer | Tool | Why |
|---|---|---|
| Hypervisor | Oracle VirtualBox | Test lab before production Hyper-V |
| OS | Ubuntu Server | LTS support, Docker-friendly |
| Containers | Docker + Compose | Declarative, portable, reproducible |
| Reverse proxy | Caddy  | Auto HTTPS, single binary |
| VPN | Tailscale | Zero-config mesh, works behind NAT |
| Firewall | UFW + Fail2ban | Default deny, brute-force protection |
| Monitoring | Uptime Kuma, Netdata *(planned)* | Uptime + metrics |
| Remote Mgmt | MeshCentral | Control endpoints, run scripts |
| Assets | Snipe-IT | Hardware, licenses, warranties |
| Ticketing | osTicket | Helpdesk queue |
| Updates | Diun | Notify when new images available |
| Secrets | Vaultwarden | Self-hosted password manager |
| Backups | restic | Encrypted, deduplicated, tested |



 What We Deferred — Caddy, Vaultwarden, Netdata

### Caddy — Deferred

**Why:** Caddy needs a domain name to obtain Let's Encrypt certificates. We do not have one yet.

**Current state:** All services are accessed via the Tailscale IP and specific ports (`http://100.114.165.59:3001`, `:8083`, `:8088`, etc.).

**When to add:** When we buy a domain and set up DNS.

### Vaultwarden — Deferred

**Why:** Will be added as the single source of truth for secrets. All service passwords will live there.

**Interim:** Passwords stored in individual notes and `.env` files.

### Netdata — Deferred

**Why:** Basic metrics currently come from Uptime Kuma. Netdata will add CPU, RAM, disk, and network dashboards.

---

## Lessons Learned

### The Big Ones

1. **Edit on the PC, add, commit and push. Pull on the server. Never edit on the server.** Every server-side edit caused drift and required cleanup. This single rule would have saved hours.

2. **Named volumes, not bind mounts, for service data.** Bind mounts leak into Git and cause permission issues.

3. **`.env` must contain the union of variables for all services.** For app + DB pairs, both sets must exist and the passwords must match.

4. **MySQL/MariaDB cache passwords on first init.** Changing them later requires deleting the volume or `ALTER USER`.

5. **Read the image's source when it behaves strangely.** One file, `docker-env.php`, explained every osTicket failure.

6. **Never disable password auth until key auth works.** And always test in a fresh session.

7. **The server is a mirror, not a source of truth.** Git is the truth.

8. **A backup you have not restored is not a backup.**

9. **Regenerate any token shared publicly.**

10. **`docker compose restart` does not reload env vars.** Use `down && up -d`.

### The Small Ones That Add Up

- `systemctl start` ≠ `systemctl enable`
- Volume names cannot contain slashes
- Compose prefixes volume names with the project name
- `git add` before `git commit`
- `sudo nano` in your home directory creates ownership issues
- `jq` is your friend for JSON
- Named volumes are backed up by restic automatically

### Meta-Lessons

- **Documentation lies when it assumes a different context.** Guides assume a domain, a reverse proxy, an admin team. Adjust for your reality.
- **When stuck, get the raw data.** `docker logs`, PHP errors, source files, `curl -v`. Guessing wastes time.
- **The error message is often correct** — read it carefully before searching.
- **It is okay to switch tools** when one is fundamentally broken. `hlhd/osticket` was broken. Switching to `cloudcogs` was the right call, even though it also had traps.

---

## What's Next

1. **Vaultwarden** — centralize secrets
2. **Netdata** — add metrics dashboards
3. **Caddy** — when a domain is available, add HTTPS and subdomains
4. **Authelia** — if GM needs browser SSO without Tailscale
5. **CrowdSec** — if services are exposed publicly
6. **Samba AD** — only if local accounts become unmanageable
7. **Monthly restore test** — prove the backup works
8. **Risk register** — document accepted risks, get sign-off from management

---




Remote Management — MeshCentral

### What We Did

1. Created `docker/meshcentral/docker-compose.yml`
2. Deployed

### Difficulties

This was one of the longest debugging sessions.

#### Difficulty 1: "Invalid origin in HTTP request"

**What happened:** After logging in, MeshCentral rejected the session with "Invalid origin in HTTP request".

**Root cause:** MeshCentral validates the browser's `Origin` header against the `cert` value in `config.json`. The default `cert` was `myserver.mydomain.com`. The browser was accessing `https://100.114.165.59:8086`. Mismatch → rejection.

**First fix attempt:** Edited `config.json` inside the volume to set `cert` to `100.114.165.59` and added `allowedOrigin: true`. Worked — but only until the volume was deleted.

#### Difficulty 2: `HOSTNAME` Env Var Ignored

**What happened:** Set `HOSTNAME=100.114.165.59` in `docker-compose.yml`. MeshCentral still logged `running on myserver.mydomain.com:443`.

**Root cause:** The `ghcr.io/ylianst/meshcentral` image **does not apply** `HOSTNAME` to certificate generation. Only `config.json` controls this.

**Fix:** Bind-mount `config.json` from the repo:

```yaml
volumes:
  - ./config.json:/opt/meshcentral/meshcentral-data/config.json:ro
```

#### Difficulty 3: `config.json` Became Empty

**What happened:** After multiple edits, `config.json` in the volume was 0 bytes. MeshCentral would not start correctly.

**Root cause:** An earlier `nano` session truncated the file. Volume state was corrupt.

**Fix:** Recreated the config from scratch, moved it into the repo, bind-mounted it. Now Git is the source of truth.

#### Difficulty 4: Port Mismatch (443 vs 8086)

**What happened:** `docker-compose.yml` mapped `8086:443`. MeshCentral thought it ran on 443. Agent installers pointed to `:443`, which was not open on the host.

**Root cause:** Internal port (443) and external port (8086) mismatched.

**Fix:** Changed everything to `8086`:

```yaml
ports:
  - "8086:8086"
  - "8087:8087"   # for HTTP redirect
```

And in `config.json`: `"port": 8086`, `"redirPort": 8087`.

### What We Learned

- **Some images ignore documented env vars.** Read the source if behavior is strange.
- **Volume-only fixes are temporary.** Move config to the repo and bind-mount it.
- **`config.json` in a volume is hidden from Git.** Bind-mounting it from the repo makes it version-controlled.
- **If the container reports the wrong port, check internal vs external mapping.**
- **"Set and forget" means the config survives a volume wipe.**

---

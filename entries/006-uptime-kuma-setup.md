### What We Did

1. Created `docker/uptime-kuma/docker-compose.yml`
2. Committed, pushed
3. Pulled on server, deployed

### Difficulties

#### Difficulty 1: Bind Mount Leaked Data into Git

**What happened:** Initially used `./data:/app/data`. Uptime Kuma's database ended up inside the repo folder, showing as untracked in `git status`.

**Root cause:** Bind mounts put data inside the working directory.

**Fix:** Migrated to a **named volume**:

```yaml
volumes:
  - uptime-kuma-data:/app/data
```

Then manually copied the existing data into the volume:

```bash
docker run --rm \
  -v uptime-kuma-data:/to \
  -v ~/it-ops-stack/docker/uptime-kuma/data:/from \
  alpine sh -c "cp -av /from/. /to/"
```

#### Difficulty 2: Cannot Delete the Old `data/` Directory

**What happened:** `rm -rf data` failed with `Permission denied`.

**Root cause:** Files were owned by the container user (`root`), not `amin`.

**Fix:** `sudo rm -rf data`.

#### Difficulty 3: Volume Name Typo

**What happened:** Compose file had `uptime-kuma/data:/app/data` — with a slash. Docker interpreted it as a bind mount and failed.

**Root cause:** Named volumes cannot contain slashes.

**Fix:** Changed to `uptime-kuma-data:/app/data`.

#### Difficulty 4: Compose-Prefixed Volume Name

**What happened:** Created volume as `uptime-kuma-data`, but Docker Compose created `uptime-kuma_uptime-kuma-data` (with project prefix). The data was in the wrong volume.

**Fix:** Migrated data between them, or used `name:` in the volume definition.

### What We Learned

- **Named volumes, not bind mounts, for service data.**
- **No slashes in volume names.**
- **Compose prefixes volume names with the project name.**
- **If git shows data files, the volume is a bind mount — convert it.**


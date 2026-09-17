The Safety Net — restic Backups

### What We Did

1. Installed restic on the **host** (not in Docker) — it needs to read `/var/lib/docker/volumes`
2. Created a restic password file at `/root/.restic/password`
3. Initialized the repository at `/backup/restic`
4. Created `/root/backup.sh` to back up Docker volumes, the Git repo, and `/etc`
5. Scheduled it with a systemd timer at 2:30 AM daily
6. Tested a restore

### Difficulties

#### Difficulty 1: `sudo -E` Not Preserving Environment

**What happened:** `sudo -E restic init` failed with `preserving the entire environment is not supported, '-E' is ignored`.

**Root cause:** Ubuntu disables `sudo -E` by default for security.

**Fix:** Pass arguments directly:
```bash
sudo restic -r /backup/restic --password-file /root/.restic/password init
```

Or use a wrapper script:
```bash
#!/bin/bash
export RESTIC_REPOSITORY=/backup/restic
export RESTIC_PASSWORD_FILE=/root/.restic/password
exec restic "$@"
```

#### Difficulty 2: Backups That Are Never Tested

**The rule:** A backup you have never restored is not a backup. It is a hope.

We tested a restore:
```bash
sudo restic restore latest --target /tmp/restore-test --include /var/lib/docker/volumes/uptime-kuma_uptime-kuma-data/_data
ls /tmp/restore-test
```

If the files were present, the backup works.

### What We Learned

- **restic belongs on the host, not in Docker.** It needs host-level access.
- **The restic password is critical.** Lose it, lose everything.
- **Store it in Vaultwarden *and* offline.**
- **Test a restore at least monthly.**
- **Use systemd timers, not cron, for modern setups.**

---

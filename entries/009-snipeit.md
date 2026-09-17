Asset Management — Snipe-IT

### What We Did

1. Created `docker/snipeit/docker-compose.yml`
2. Created `.env` with both `MYSQL_*` and `DB_*` variables
3. Deployed
4. Completed the setup wizard

### Difficulties

#### Difficulty 1: Access Denied for User 'snipeit'

**What happened:** The Snipe-IT wizard showed "Access denied for user 'snipeit'@'172.18.0.5'".

**Root cause:** Two layers:
1. MariaDB reads `MYSQL_PASSWORD` **only on first initialization**. Once the DB is created, changing the env var has no effect.
2. The `.env` was missing `DB_*` variables. The app was sending a blank password.

**Fix:**
1. Added `DB_PASSWORD` and `DB_USERNAME` and `DB_DATABASE` to `.env`, matching `MYSQL_*`.
2. Deleted the DB volume to force re-initialization with the correct password.

```bash
docker compose down
docker volume rm snipeit_snipeit-db-data
docker compose up -d
```

### What We Learned

- **MariaDB / MySQL cache passwords from first init.** Changing them later requires deleting the volume or running `ALTER USER`.
- **`.env` must contain the union of variables for both services.**
- **The app reads `DB_*`; the DB reads `MYSQL_*`. They must be set and matching.**
- **This same lesson will appear with osTicket.**

---


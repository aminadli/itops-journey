The Boss Fight — osTicket

This was the longest and most painful phase. Three different images, multiple entrypoint bugs, one env-var trap. Every failure was instructive.

### Attempt 1: `lliwi/osticket:1.18.1`

**Error:** `pull access denied for lliwi/osticket`.

**Cause:** The tag `1.18.1` was removed from Docker Hub.

**Attempted Fix:** Switched to `1.18.3`. Same error — Docker Hub returned "pull access denied" for a tag that existed.

### Attempt 2: `hlhd/osticket`

**Error 1:** `[entrypoint] DB not reachable, giving up` — but MySQL was up and reachable.

**Cause:** The `hlhd` image is Alpine-based. Its entrypoint uses `bash` for the DB wait loop. Alpine has no `bash`. The check **always** failed, even with the DB up.

Proven by:
```bash
docker exec osticket nc -zv osticket-db 3306
# → osticket-db (172.18.0.7:3306) open
```

**Error 2:** Wizard rejected every valid submission with "Missing or invalid data". The HTML showed 7 fields empty even though they were submitted.

**Cause:** `hlhd` also had a broken web installer.

### Attempt 3: `cloudcogs/osticket`

**Error:** Same wizard issue — 7 fields always empty on POST.

**Root Cause (the big one):**

The image contains `/var/www/html/setup/docker-env.php`:

```php
<?php
$_POST['s'] = 'install';
$_POST['name'] = getenv('HELPDESK_NAME') ?: '';
$_POST['email'] = getenv('DEFAULT_EMAIL') ?: '';
$_POST['dbhost'] = getenv('CONFIG_DBHOST') ?: 'localhost';
$_POST['dbname'] = getenv('MARIADB_DATABASE') ?: '';
$_POST['dbuser'] = getenv('MARIADB_USER') ?: '';
$_POST['dbpass'] = getenv('MARIADB_PASSWORD') ?: '';
$_POST['prefix'] = getenv('TABLE_PREFIX') ?: 'ost_';
$_POST['admin_email'] = getenv('ADMIN_EMAIL') ?: '';
$_POST['fname'] = getenv('ADMIN_FIRSTNAME') ?: '';
$_POST['lname'] = getenv('ADMIN_LASTNAME') ?: '';
$_POST['username'] = getenv('ADMIN_USERNAME') ?: '';
$_POST['passwd'] = getenv('ADMIN_PASSWORD') ?: '';
```

**This file overwrites `$_POST` with environment variables on every request.** The web form is **ignored entirely**. The installer reads only env vars.

The env var names we had were wrong:

| `docker-env.php` reads | Our compose had |
|---|---|
| `HELPDESK_NAME` | `INSTALL_NAME` |
| `DEFAULT_EMAIL` | `INSTALL_EMAIL` |
| `MARIADB_DATABASE` | `CONFIG_DBNAME` |
| `MARIADB_USER` | `CONFIG_DBUSER` |
| `MARIADB_PASSWORD` | `CONFIG_DBPASS` |
| `TABLE_PREFIX` | `CONFIG_TABLE_PREFIX` |
| `ADMIN_USERNAME` | `ADMIN_USER` |
| `ADMIN_PASSWORD` | `ADMIN_PASS` |

The 7 failing form fields matched exactly the 7 missing env vars.

**Final Fix:** Updated `docker-compose.yml` to use the correct env var names, made `DEFAULT_EMAIL` different from `ADMIN_EMAIL` (osTicket enforces this), and wiped the volumes.

The install then completed in one click.

### Additional Fix: `ost-config.php` Permissions

The installer required `ost-config.php` to exist and be writable. We created it from the sample:

```bash
docker exec osticket cp /var/www/html/include/ost-sampleconfig.php /var/www/html/include/ost-config.php
docker exec -u root osticket chmod 666 /var/www/html/include/ost-config.php
```

After successful install, we locked it down:

```bash
docker exec -u root osticket chmod 644 /var/www/html/include/ost-config.php
docker exec -u root osticket chmod 755 /var/www/html/include/
```

### What We Learned

- **Not all Docker images behave the way their documentation claims.**
- **Read the image's source when the behavior is unexplainable.** One file (`docker-env.php`) held the entire answer.
- **Different maintainers use different env var names** for the same product.
- **Never trust a form to be the source of truth.**
- **MySQL/MariaDB cache passwords on first init.**
- **Wipe volumes when the schema or env vars change.**
- **The webpage's "Congratulations" message is accurate — but it is not the whole story.** The env vars are the real setup.

---


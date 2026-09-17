Update Notifier — Diun

### What We Did

1. Created a Telegram bot via `@BotFather`
2. Got the bot token
3. Got the chat ID via `@userinfobot`
4. Created `docker/diun/docker-compose.yml` with env var references
5. Created `.env.example` and `.env` on the server

### Difficulties

#### Difficulty 1: Duplicate YAML Keys

**What happened:** `docker compose up -d` failed with:
```
yaml: construct errors: line 45: mapping key "networks" already defined at line 23
```

**Root cause:** Two compose files were pasted on top of each other. Two `services:`, two `networks:`, two `volumes:` blocks.

**Fix:** Deleted the file and pasted a single clean version.

#### Difficulty 2: Chat ID vs Bot ID Confusion

**What happened:** Diun kept failing with:
- `Unauthorized` (wrong token)
- `Forbidden: the bot can't send messages to the bot` (chat ID equal to bot ID)

**Root cause:**
- The `Unauthorized` error was because the token in `.env` was the old one — I had regenerated the token in BotFather.
- The `Forbidden` error was because the chat ID was the **bot ID** (number before the colon in the token), not my personal Telegram user ID.

**Fix:**
1. Updated `DIUN_TELEGRAM_TOKEN` with the new token
2. Used `@userinfobot` to find the actual chat ID
3. Verified with `curl https://api.telegram.org/bot<TOKEN>/getUpdates`
4. The correct value is `"chat": {"id": NNNNNNNNN}` — not `"from": {"id": ...}` and not `"update_id"`

#### Difficulty 3: After Changing `.env`, Container Did Not Reload

**What happened:** Changed `DIUN_TELEGRAM_TOKEN` in `.env`, ran `docker compose restart` — token did not update.

**Root cause:** `docker compose restart` does **not** reload environment variables. Env vars are read at container **creation** time.

**Fix:**
```bash
docker compose down
docker compose up -d
```

### What We Learned

- **Diun does not auto-update.** It only notifies.
- **`.env` is read at container creation, not restart.**
- **Env vars in `docker-compose.yml` use `${VAR}` substitution from `.env`.**
- **Telegram bot tokens and chat IDs are different things.**
- **Always regenerate a leaked token.**

---

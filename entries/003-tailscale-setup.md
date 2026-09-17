
### What We Did

1. Installed Tailscale on the server
2. Installed Tailscale on the PC
3. Logged both into the same account
4. Verified connection with `tailscale status` and `ping`

### Difficulties

#### Difficulty 1: Tailscale Not Installed on PC

**What happened:** Installed Tailscale on the server, then tried to `ping 100.x.y.z` from the PC. It failed.

**Root cause:** Tailscale is a **peer-to-peer mesh**. Both ends need it. The server had a Tailscale IP, but the PC could not route to it.

**Fix:** Installed Tailscale on the PC and logged into the same account.

#### Difficulty 2: `tailscale` Command Not Found on Windows

**What happened:** After installing via `winget`, `tailscale status` in PowerShell said "not recognized".

**Root cause:** The installer puts the binary at `C:\Program Files\Tailscale\tailscale.exe`, but does not add it to PATH automatically in all cases.

**Fix:** Added the path manually or used the full path.


### What We Learned

- **Tailscale is a two-sided connection.** Both devices need it.




### What We Did

1. Generated an SSH key on the PC 
2. Copied the public key to the server with `ssh-copy-id`
3. Edited `/etc/ssh/sshd_config.d/hardening.conf`
4. Installed Fail2ban
5. Enabled UFW

### The Hardening Config

```
Port 22
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
LoginGraceTime 20
X11Forwarding no
AllowTcpForwarding no
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers amin
```

### Difficulties

#### Difficulty 1: Locked Out of SSH

**What happened:** Applied the hardening config. Immediately after, SSH failed with `Permission denied`. The console still worked.

**Root cause:** Two problems combined:

1. `AllowUsers admin` — but the actual user was `amin`. This alone locked out the only account. My mistake is to just copy online tutorial without full understanding of line by line. user in the command must match our created user.
2. `PasswordAuthentication no` — but the key was never installed on the PC.

**Why the key was missing:** I had generated the SSH key **on the server**, not on the PC. The public key was in the server's own `~/.ssh/authorized_keys` (useless), and the private key was on the wrong machine. At this time, SSH is still a new concept to me and i have misunderstanding how the ssh key work and which device generate the key and which device keep the key

**In simpler terms, endpoint generate public key and we copy the key and paste to the server so the server recognize which endpoint to trust, because the endpoint want to remote into the computer using ssh

**Fix:**
1. Opened the VM console
2. Edited `hardening.conf`: set `PasswordAuthentication yes`, changed `AllowUsers amin`
3. Ran `sudo sshd -t` (validate), then `sudo systemctl restart sshd`
4. Logged in via password
5. Generated a fresh key **on the PC**
6. Copied the public key to the server with `ssh-copy-id`
7. Tested key login from a **new** session
8. Only then re-hardened (`PasswordAuthentication no`, `AllowUsers amin`)

#### Difficulty 2: sshd Not Enabled on Boot

**What happened:** After a reboot, SSH stopped working. Ping worked, but SSH timed out.

**Root cause:** `sshd` was **started** but not **enabled**. Started services do not come back after reboot.

**Fix:**
```bash
sudo systemctl enable sshd
sudo systemctl start sshd
```

**Lesson:** Always use `systemctl enable --now` for services that must survive reboots.

### What We Learned

- **Never blindly paste code from online tutorial without line by line understanding
- **Never disable password auth before testing key auth in a fresh session.**
- **Never set `AllowUsers` to a name you have not tested.**
- **Always run `sshd -t` before restarting.**
- **Keep the old session open when changing SSH config.**
- **Generate keys where you connect FROM, not where you connect TO.**
- **`systemctl start` ≠ `systemctl enable`.**

This was the first real lockout, and it taught more about SSH than any tutorial could.

---


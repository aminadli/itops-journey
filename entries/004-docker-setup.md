
### What We Did

1. Installed Docker via the official convenience script
2. Added user to the `docker` group
3. Created a shared Docker network (`shared_network`)
4. Verified with `docker run hello-world`

### Difficulties

| Problem | Fix |
|---|---|
| `permission denied` on Docker commands | `sudo usermod -aG docker $USER` then `newgrp docker` |
| Network missing error later | `docker network create shared_network` |

### What We Learned

The `shared_network` is essential. Every service joins it. Caddy will use it later to reach backends by name. Create it once and forget it.


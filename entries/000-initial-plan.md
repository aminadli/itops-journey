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


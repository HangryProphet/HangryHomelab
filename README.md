# HangryHomelab

HangryHomelab is my personal Proxmox-based homelab for learning and practicing
networking, virtualization, Linux administration, storage, monitoring,
self-hosted services, and infrastructure troubleshooting.

The goal is to build practical hands-on experience beyond classroom labs and
use the environment as both a learning platform and a documented portfolio
project.

## Current Hardware

- **Host:** Lenovo ThinkCentre M910q Tiny
- **CPU:** Intel Core i5-7500T
- **RAM:** 16 GB DDR4
- **Primary Storage:** 240 GB SATA SSD
- **Bulk Storage:** 500 GB USB HDD
- **Network:** Intel Ethernet Connection (2) I219-V, 1 Gbps full-duplex
- **Hypervisor:** Proxmox VE

More details are available in [`docs/hardware.md`](docs/hardware.md).

## Current Status

- [x] Proxmox VE installed and configured
- [x] Linux LXC deployment
- [x] External HDD integrated as `bulk-hdd`
- [x] Minecraft server deployed with Crafty Controller
- [x] Playit.gg configured for CGNAT tunneling
- [x] Minecraft environment backed up and archived
- [x] Minecraft LXC retired to reclaim SSD space
- [ ] AdGuard Home
- [ ] Jellyfin
- [ ] Uptime Kuma
- [ ] Windows and Linux virtual machines
- [ ] VLAN segmentation
- [ ] Routing and firewall labs
- [ ] Network monitoring
- [ ] Backup and recovery testing

## Planned Lab Areas

### Networking

- DNS
- DHCP
- Static addressing
- VLANs
- Inter-VLAN routing
- NAT
- Firewall rules
- VPNs and tunnels
- Network troubleshooting

### Systems and Virtualization

- Proxmox VE
- Linux containers
- Virtual machines
- Linux administration
- systemd services
- Storage and mount management
- Backups and restoration
- Monitoring and logging

### Self-Hosted Services

Planned or previously deployed services include:

- AdGuard Home
- Jellyfin
- Uptime Kuma
- Minecraft / Crafty Controller
- Playit.gg

## Documentation

- [`Hardware`](docs/hardware.md)
- [`Minecraft Archival`](docs/minecraft-archive.md)

More documentation will be added as the lab grows.

## Why I Built This

I built HangryHomelab as a practical environment for experimenting with
networking and infrastructure concepts while documenting what I learn,
including both successful deployments and troubleshooting incidents.

The project is intended to grow over time as I work toward networking,
infrastructure, and systems-related roles.

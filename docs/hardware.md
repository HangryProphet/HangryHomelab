# Hardware

This page documents the physical hardware used for HangryHomelab.

## Proxmox Host

- **Model:** Lenovo ThinkCentre M910q Tiny
- **CPU:** Intel Core i5-7500T
- **RAM:** 16 GB
- **Hypervisor:** Proxmox VE
- **Network:** - **Network:** Intel Ethernet Connection (2) I219-V, 1 Gbps full-duplex

## Storage

### Primary SSD

- **Model:** Skihotar A320
- **Capacity:** 240 GB
- **Type:** 2.5-inch SATA SSD
- **Purpose:**
  - Proxmox VE
  - Active LXCs
  - Active VMs
  - Services that benefit from faster storage

### Bulk Storage HDD

- **Model:** Seagate ST500LM021-1KJ152
- **Capacity:** 500 GB
- **Speed:** 7200 RPM
- **Connection:** USB 3.0 SATA enclosure
- **Proxmox Storage Name:** `bulk-hdd`
- **Purpose:**
  - Proxmox backups
  - Archived services
  - Media storage
  - ISO images
  - General bulk storage

The HDD is intentionally not used for active VM or LXC disks because
the SSD provides better latency and performance.

## Network Equipment

- PLDT fiber router / ONT
- Ethernet switch
- Cat6 Ethernet cabling

## Storage Strategy

The lab separates active workloads from bulk and backup storage.

```text
240 GB SSD
├── Proxmox VE
├── LXCs
├── VMs
└── Active services

500 GB HDD
├── Backups
├── Archives
├── Media
└── Bulk storage

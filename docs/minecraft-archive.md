# Minecraft Server Archive

The Minecraft server was one of the first services I hosted in my Proxmox
homelab. It ran inside a Debian LXC with Crafty Controller and Playit.gg.

I decided to retire the server for now so I could reclaim SSD space and use
the homelab more heavily for networking and infrastructure projects.

## Original Setup

- Debian LXC
- 3 CPU cores
- 10 GB RAM
- 67 GB root disk
- Crafty Controller
- Playit.gg
- Java
- Modded Minecraft server

The LXC was stored on the primary SSD.

## Backup Plan

Before deleting the container, I created two separate backups.

### Full LXC Backup

A Proxmox VZDump backup was created and stored on `bulk-hdd`.

The backup uses Zstandard compression and contains the complete LXC
environment, allowing the entire Minecraft server setup to be restored later.

This includes:

- Debian
- Crafty Controller
- Java
- Playit.gg
- Minecraft files
- Configuration
- Installed services

### Minecraft Server Archive

I also created a separate compressed archive of Crafty's `servers` directory.

This contains the Minecraft-specific files such as the world, mods,
configuration files, and server settings.

The archive was stored under:

`bulk-hdd/minecraft-archive/`

This gives me the option to recover only the Minecraft server without
restoring the entire old LXC.

## Verification

Before removing the original container:

- The Proxmox backup completed successfully.
- The Minecraft archive was checked with `tar`.
- The LXC was confirmed to have no additional mount points containing
  external server data.

After verifying the backups, the original Minecraft LXC was removed from
the SSD.

## Recovery

If I want the complete environment back, I can restore the VZDump backup
through Proxmox.

If I only need the Minecraft server data, I can extract the separate
`crafty-servers` archive and import the files into another server environment.

## What I Learned

- Proxmox LXC backups and restoration
- VZDump
- Zstandard compression
- Linux filesystem mounting
- Creating and verifying TAR archives
- Separating active storage from backup storage
- Planning for service recovery before deleting workloads

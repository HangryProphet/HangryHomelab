# Palworld Dedicated Server

This document summarizes the Palworld server running inside my Proxmox homelab.

The server is designed for a small group of around five players and runs inside a lightweight Debian LXC.

## Architecture

```text
Proxmox VE
└── Debian 13 LXC
    ├── SteamCMD
    ├── Wine
    │   └── Windows Palworld Dedicated Server
    ├── UE4SS
    ├── PalSchema
    ├── Crafting100x
    ├── Quality of Life Schema
    └── SSH / SCP
```

The active server runs from SSD storage. Backups are planned for the external `bulk-hdd`.

## LXC Resources

- **CT ID:** 101
- **Hostname:** `palworld`
- **OS:** Debian 13
- **CPU:** 4 vCPU
- **RAM:** 10 GB
- **Swap:** 2 GB
- **Disk:** 50 GB
- **Storage:** `local-lvm`
- **Network Bridge:** `vmbr0`
- **Current LAN IP:** `192.168.1.44`
- **Palworld Port:** `8211/UDP`

## Why Windows Palworld on Linux?

The server was originally installed using the native Linux Palworld dedicated server.

The final mod stack required the Windows UE4SS and PalSchema ecosystem, so instead of using a full Windows VM, the architecture was changed to:

```text
Debian LXC
└── Wine
    └── Windows Palworld Dedicated Server
```

This kept the deployment lightweight while providing better compatibility with the planned mods.

## Software Stack

### SteamCMD

SteamCMD is used to download and update the Palworld dedicated server.

Palworld Dedicated Server App ID:

```text
2394010
```

The Windows server is installed under:

```text
/home/steam/palworld-win
```

### Wine

Wine 10 is used to run the Windows Palworld server inside Debian.

Wine prefix:

```text
/home/steam/palworld-win/.wine
```

The Visual C++ 2022 runtime is installed inside the prefix using Winetricks.

### UE4SS

UE4SS provides the mod-loading layer required by PalSchema.

Important files:

```text
/home/steam/palworld-win/Pal/Binaries/Win64/dwmapi.dll
/home/steam/palworld-win/Pal/Binaries/Win64/ue4ss/
```

The server must be launched with:

```bash
WINEDLLOVERRIDES="dwmapi=n,b"
```

so Wine loads the UE4SS proxy DLL.

### PalSchema

Installed version:

```text
0.6.71
```

Location:

```text
/home/steam/palworld-win/Pal/Binaries/Win64/ue4ss/Mods/PalSchema/
```

## Installed Mods

### Crafting100x

Location:

```text
ue4ss/Mods/PalSchema/mods/Crafting100x/
```

The mod changes crafting output quantities for selected items and can be tuned later.

### Quality of Life Schema

Current Workshop ID:

```text
3798638830
```

UE4SS location:

```text
ue4ss/Mods/QoLSchema/
```

The mod successfully generated:

```text
SchemaQoLConfig.user.json
SchemaQoLAdvanced.user.json
```

Current default features include changes to:

- Work speed
- Base range
- Storage
- Fishing
- Pal revive time
- Expeditions
- Research
- Condensation
- Palbox capacity
- Inventory pouches

## Important Configuration Files

### Palworld Server

```text
/home/steam/palworld-win/Pal/Saved/Config/WindowsServer/PalWorldSettings.ini
```

Controls vanilla server settings such as XP rate, capture rate, damage, spawn rates, gathering, passwords, player limits, and base/guild limits.

### Quality of Life Schema

```text
/home/steam/palworld-win/Pal/Binaries/Win64/ue4ss/Mods/QoLSchema/SchemaQoLConfig.user.json
/home/steam/palworld-win/Pal/Binaries/Win64/ue4ss/Mods/QoLSchema/SchemaQoLAdvanced.user.json
```

### Crafting100x

```text
/home/steam/palworld-win/Pal/Binaries/Win64/ue4ss/Mods/PalSchema/mods/Crafting100x/raw/
```

### PalSchema

```text
/home/steam/palworld-win/Pal/Binaries/Win64/ue4ss/Mods/PalSchema/config/config.json
```

### UE4SS

```text
/home/steam/palworld-win/Pal/Binaries/Win64/ue4ss/UE4SS-settings.ini
/home/steam/palworld-win/Pal/Binaries/Win64/ue4ss/Mods/mods.txt
```

## Networking Test

The server was tested locally using:

```text
192.168.1.44:8211
```

A Palworld client on the same LAN successfully connected, created a character, saved progress, and retained the same world and character data after a server restart.

## Notable Troubleshooting

### Native Linux to Wine

The native Linux server worked, but the required mod stack was better supported by the Windows server. The final solution was to keep Debian and run the Windows server through Wine rather than deploy a full Windows VM.

### Wine Warnings

Wine produced several OLE, RPC, Xvfb, and Steam networking warnings during setup. Each layer was tested independently, and the server still reached port `8211`, accepted clients, and preserved save data.

### UE4SS Injection

UE4SS required Wine to explicitly prefer the native proxy DLL:

```bash
WINEDLLOVERRIDES="dwmapi=n,b"
```

### QoLSchema

Quality of Life Schema initially did not load. It was enabled in:

```text
ue4ss/Mods/mods.txt
```

using:

```text
QoLSchema : 1
```

After this, the mod generated its user configuration files and loaded correctly.

### Server Shutdown

`Ctrl+C` does not reliably stop the current Wine/Xvfb process chain. A proper systemd service is planned.

## Current Status

- [x] Debian LXC deployed
- [x] SteamCMD installed
- [x] Windows Palworld server running through Wine
- [x] LAN connectivity confirmed
- [x] Save persistence confirmed
- [x] UE4SS installed
- [x] PalSchema installed
- [x] Crafting100x installed
- [x] Quality of Life Schema installed
- [x] SSH/SCP file transfer configured
- [ ] Final gameplay configuration
- [ ] Admin password and admin workflow
- [ ] systemd service
- [ ] Playit.gg tunnel
- [ ] External multiplayer test
- [ ] Backup and restore workflow

## Next Steps

1. Finalize vanilla and mod balance settings.
2. Create a systemd service for reliable server management.
3. Configure Playit.gg for external access through CGNAT.
4. Test multiplayer from outside the LAN.
5. Create full LXC and Palworld-only backups on `bulk-hdd`.

For installation, maintenance, troubleshooting, and recovery commands, see [`palworld-runbook.md`](palworld-runbook.md).

# Palworld Server Runbook

This runbook contains the practical commands and paths used to deploy and operate the Palworld server documented in [`palworld-server.md`](palworld-server.md).

## Current LXC

```text
CT ID: 101
Hostname: palworld
CPU: 4 vCPU
RAM: 10 GB
Swap: 2 GB
Disk: 50 GB
LAN IP: 192.168.1.44
```

## Debian Preparation

As `root`:

```bash
apt update && apt full-upgrade -y
apt install -y curl tar ca-certificates lib32gcc-s1
```

Create the Steam account:

```bash
adduser --disabled-password --gecos "" steam
id steam
su - steam
```

## SteamCMD

```bash
mkdir -p ~/Steam
cd ~/Steam
curl -L -o steamcmd_linux.tar.gz https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz
tar -xzf steamcmd_linux.tar.gz
./steamcmd.sh
```

Exit SteamCMD with:

```text
quit
```

## Native Linux Palworld

Originally downloaded with:

```bash
cd ~/Steam

./steamcmd.sh +login anonymous +app_update 2394010 validate +quit
```

A first-run `Missing configuration` error was encountered. Retrying the same command succeeded.

Native Linux server location:

```text
/home/steam/Steam/steamapps/common/PalServer
```

## Wine Installation

As `root`:

```bash
dpkg --add-architecture i386
apt update

apt install -y wine wine64 wine32 libwine libwine:i386 fonts-wine cabextract xvfb winbind
```

Verify:

```bash
wine --version
```

Known working result:

```text
wine-10.0
```

## Windows Palworld Server

As `steam`:

```bash
mkdir -p ~/palworld-win
cd ~/Steam

./steamcmd.sh +@sSteamCmdForcePlatformType windows +force_install_dir /home/steam/palworld-win +login anonymous +app_update 2394010 validate +quit
```

Verify:

```bash
ls ~/palworld-win
ls ~/palworld-win/Pal/Binaries/Win64
```

Important files:

```text
PalServer.exe
PalServer-Win64-Shipping-Cmd.exe
```

## Wine Prefix

```bash
WINEPREFIX=/home/steam/palworld-win/.wine xvfb-run -a wineboot -u
```

Verify:

```bash
ls ~/palworld-win/.wine
```

Test Wine:

```bash
WINEPREFIX=/home/steam/palworld-win/.wine xvfb-run -a wine cmd /c ver
```

Known working output:

```text
Microsoft Windows 10.0.19043
```

## Visual C++ Runtime

As root:

```bash
apt install -y winetricks
```

As `steam`:

```bash
WINEPREFIX=/home/steam/palworld-win/.wine xvfb-run -a winetricks -q vcrun2022
```

Verify:

```bash
WINEPREFIX=/home/steam/palworld-win/.wine winetricks list-installed
```

Expected:

```text
vcrun2022
```

## First Windows Server Boot

```bash
cd ~/palworld-win/Pal/Binaries/Win64

WINEPREFIX=/home/steam/palworld-win/.wine xvfb-run -a wine PalServer-Win64-Shipping-Cmd.exe
```

Expected:

```text
Running Palworld dedicated server on :8211
```

## Palworld Configuration

Copy the default configuration:

```bash
cp ~/palworld-win/DefaultPalWorldSettings.ini ~/palworld-win/Pal/Saved/Config/WindowsServer/PalWorldSettings.ini
```

Example server name:

```bash
sed -i 's/ServerName="[^"]*"/ServerName="HangryPalworld"/' ~/palworld-win/Pal/Saved/Config/WindowsServer/PalWorldSettings.ini
```

Example player limit:

```bash
sed -i 's/ServerPlayerMaxNum=[0-9]*/ServerPlayerMaxNum=8/' ~/palworld-win/Pal/Saved/Config/WindowsServer/PalWorldSettings.ini
```

Example XP adjustment:

```bash
sed -i 's/ExpRate=[0-9.]*/ExpRate=0.5/' ~/palworld-win/Pal/Saved/Config/WindowsServer/PalWorldSettings.ini
```

Verify:

```bash
grep -o 'ExpRate=[^,)]*' ~/palworld-win/Pal/Saved/Config/WindowsServer/PalWorldSettings.ini
```

## Check LXC IP

From the Proxmox host:

```bash
pct exec 101 -- ip -br addr
```

Current LAN address:

```text
192.168.1.44
```

Local connection:

```text
192.168.1.44:8211
```

## UE4SS

Go to:

```bash
cd ~/palworld-win/Pal/Binaries/Win64
```

Download:

```bash
curl -L -o UE4SS-Palworld.zip https://github.com/Okaetsu/RE-UE4SS/releases/download/experimental-palworld/UE4SS-Palworld.zip
```

Extract:

```bash
unzip -o UE4SS-Palworld.zip
```

Verify:

```bash
ls -d dwmapi.dll ue4ss
```

## Launch With UE4SS

```bash
cd ~/palworld-win/Pal/Binaries/Win64

WINEDLLOVERRIDES="dwmapi=n,b" WINEPREFIX=/home/steam/palworld-win/.wine xvfb-run -a wine PalServer-Win64-Shipping-Cmd.exe
```

UE4SS log:

```text
/home/steam/palworld-win/Pal/Binaries/Win64/ue4ss/UE4SS.log
```

Check:

```bash
tail -n 30 ue4ss/UE4SS.log
```

## PalSchema

Check latest release:

```bash
curl -s https://api.github.com/repos/Okaetsu/PalSchema/releases/latest | grep browser_download_url
```

Known working version:

```text
0.6.71
```

Download:

```bash
curl -L -o PalSchema_0.6.71.zip https://github.com/Okaetsu/PalSchema/releases/download/0.6.71/PalSchema_0.6.71.zip
```

Extract:

```bash
unzip -o PalSchema_0.6.71.zip -d ~/palworld-win/Pal/Binaries/Win64/ue4ss/Mods/
```

Verify:

```bash
ls ue4ss/Mods/PalSchema
```

Check load:

```bash
grep -i "PalSchema" ue4ss/UE4SS.log | tail -n 30
```

Known working line:

```text
[PalSchema] PalSchema v0.6.71 by Okaetsu loaded.
```

## Crafting100x

```bash
mkdir -p ~/crafting100x-temp
cd ~/crafting100x-temp

curl -L -o Crafting100x.zip https://github.com/YYuX-1145/Palworld-Crafting100x/archive/refs/heads/main.zip

unzip -o Crafting100x.zip

mkdir -p ~/palworld-win/Pal/Binaries/Win64/ue4ss/Mods/PalSchema/mods/Crafting100x

cp -a Palworld-Crafting100x-main/PalSchema/. ~/palworld-win/Pal/Binaries/Win64/ue4ss/Mods/PalSchema/mods/Crafting100x/
```

Verify:

```bash
find ~/palworld-win/Pal/Binaries/Win64/ue4ss/Mods/PalSchema/mods/Crafting100x -maxdepth 2 -type f
```

Important configuration folder:

```text
ue4ss/Mods/PalSchema/mods/Crafting100x/raw/
```

Check load:

```bash
grep -i "Crafting100x" ue4ss/UE4SS.log | tail -n 30
```

## SSH / SCP

As root:

```bash
apt install -y openssh-server
systemctl enable --now ssh
passwd steam
```

Test from Windows:

```powershell
ssh steam@192.168.1.44
```

## Quality of Life Schema

Current Workshop ID:

```text
3798638830
```

Current Windows Workshop path:

```text
E:\SteamLibrary\steamapps\workshop\content\1623730\3798638830
```

If Steam does not immediately create the Workshop directory after subscribing, launch Palworld once and check again.

Transfer from Windows:

```powershell
scp -r "E:\SteamLibrary\steamapps\workshop\content\1623730\3798638830" steam@192.168.1.44:/home/steam/
```

Create destination:

```bash
mkdir -p ~/palworld-win/Pal/Binaries/Win64/ue4ss/Mods/QoLSchema
```

Copy Scripts:

```bash
cp -a ~/3798638830/Scripts ~/palworld-win/Pal/Binaries/Win64/ue4ss/Mods/QoLSchema/
```

Copy Data:

```bash
cp -a ~/3798638830/Data ~/palworld-win/Pal/Binaries/Win64/ue4ss/Mods/QoLSchema/
```

Merge PalSchema files:

```bash
cp -a ~/3798638830/PalSchema/. ~/palworld-win/Pal/Binaries/Win64/ue4ss/Mods/PalSchema/
```

Enable in:

```text
ue4ss/Mods/mods.txt
```

Add:

```text
QoLSchema : 1
```

Quick method:

```bash
echo 'QoLSchema : 1' >> ue4ss/Mods/mods.txt
```

After a successful launch, verify:

```bash
ls ue4ss/Mods/QoLSchema
```

Expected generated files include:

```text
SchemaQoLConfig.user.json
SchemaQoLAdvanced.user.json
```

Check log:

```bash
grep -i "QoLSchema\|Loaded user config" ue4ss/UE4SS.log | tail -n 40
```

Known working lines include:

```text
Starting Lua mod 'QoLSchema'
[Lua] [QoLSchema] Loaded.
[PalSchema] Loading mod: QoLSchema
```

QoLSchema may require one additional server restart after generating PalSchema values.

## Important Config Files

Find the major files:

```bash
find ~/palworld-win -type f \( -name "PalWorldSettings.ini" -o -name "SchemaQoLConfig.user.json" -o -name "SchemaQoLAdvanced.user.json" -o -path "*/Crafting100x/raw/*.json" -o -path "*/PalSchema/config/config.json" -o -name "UE4SS-settings.ini" -o -name "mods.txt" \) -print
```

### Vanilla Palworld

```text
/home/steam/palworld-win/Pal/Saved/Config/WindowsServer/PalWorldSettings.ini
```

### QoLSchema

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

## Current Manual Start

```bash
cd ~/palworld-win/Pal/Binaries/Win64

WINEDLLOVERRIDES="dwmapi=n,b" WINEPREFIX=/home/steam/palworld-win/.wine xvfb-run -a wine PalServer-Win64-Shipping-Cmd.exe
```

Expected:

```text
Running Palworld dedicated server on :8211
```

## Current Manual Stop

`Ctrl+C` is unreliable with the current Wine/Xvfb process chain.

From the Proxmox host:

```bash
pct exec 101 -- pkill -INT -f PalServer-Win64-Shipping-Cmd.exe
```

Check:

```bash
pct exec 101 -- ps aux | grep -i PalServer
```

If still running:

```bash
pct exec 101 -- pkill -TERM -f PalServer-Win64-Shipping-Cmd.exe
```

Avoid `SIGKILL` unless necessary.

## Troubleshooting

### SteamCMD: Missing Configuration

Symptom:

```text
ERROR! Failed to install app '2394010' (Missing configuration)
```

Observed fix: retry the same SteamCMD command once.

### Wine OLE / RPC Warnings

Verify Wine instead of assuming failure:

```bash
WINEPREFIX=/home/steam/palworld-win/.wine xvfb-run -a wine cmd /c ver
```

### Xvfb Connection Messages

Messages such as:

```text
X connection to :99 broken
```

usually appear when Xvfb exits and are not automatically fatal.

### Steam Networking Warning

A `GetAdaptersAddresses` assertion appeared under Wine.

The server was still considered healthy after verifying port `8211`, LAN connection, character creation, world save, and persistence after restart.

### QoLSchema Does Not Load

Check:

```bash
grep -i "QoLSchema" ue4ss/UE4SS.log
```

Confirm:

```text
QoLSchema : 1
```

exists in:

```text
ue4ss/Mods/mods.txt
```

## Planned systemd Service

Not configured yet.

Goal:

```bash
systemctl start palworld
systemctl stop palworld
systemctl restart palworld
systemctl status palworld
```

The service should run as `steam` and use:

```text
WINEPREFIX=/home/steam/palworld-win/.wine
WINEDLLOVERRIDES=dwmapi=n,b
```

## Planned Playit.gg

Not configured yet.

Target:

```text
Protocol: UDP
Local port: 8211
```

## Planned Backups

### Full LXC

Use Proxmox VZDump and store on:

```text
bulk-hdd
```

### Palworld-Only

Archive at minimum:

```text
/home/steam/palworld-win/Pal/Saved/
/home/steam/palworld-win/Pal/Binaries/Win64/ue4ss/
```

Also preserve `PalWorldSettings.ini`, QoLSchema user configs, and Crafting100x JSON files.

## Useful Verification Commands

```bash
free -h
nproc
df -h /
ip -br addr
ps aux | grep -i PalServer
```

From Proxmox:

```bash
pct exec 101 -- ip -br addr
```

UE4SS:

```bash
tail -n 30 ~/palworld-win/Pal/Binaries/Win64/ue4ss/UE4SS.log
```

PalSchema:

```bash
grep -i "PalSchema" ~/palworld-win/Pal/Binaries/Win64/ue4ss/UE4SS.log | tail -n 30
```

Crafting100x:

```bash
grep -i "Crafting100x" ~/palworld-win/Pal/Binaries/Win64/ue4ss/UE4SS.log | tail -n 30
```

QoLSchema:

```bash
grep -i "QoLSchema\|Loaded user config" ~/palworld-win/Pal/Binaries/Win64/ue4ss/UE4SS.log | tail -n 40
```

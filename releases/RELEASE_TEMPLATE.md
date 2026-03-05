# Docker Migration Tool v0.8.0

## Highlights

- Backup/Restore von Docker-Containern inkl.:
  - Container-Config
  - Images
  - Named Volumes
  - Bind-Mount Daten
- Dry-Run + Plan-Modus
- Archiv-Validierung (`verify`, `backup --verify`)
- Restore mit erweiterten Optionen:
  - Labels, User, Workdir
  - Network Mode + Multi-Network Reconnect
  - Extra Hosts, CapAdd/CapDrop
- Optionales Nachziehen fehlender Images: `--pull-missing-images`
- Persistierte Netzwerk-Metadaten (Driver/IPAM Subnet/Gateway)

## Binaries

- `dockermigrate-linux-amd64`
- `dockermigrate-darwin-amd64`
- `dockermigrate-darwin-arm64`
- `dockermigrate-windows-amd64.exe`

## Checksums (SHA256)

Siehe Datei `SHA256SUMS` im selben Release.

## Quick Start

### Linux/macOS

```bash
chmod +x dockermigrate-<os>-amd64
./dockermigrate-<os>-amd64 version
```

### Windows (PowerShell)

```powershell
.\dockermigrate-windows-amd64.exe version
```

## Beispiel: Backup + Restore

```bash
# Backup
./dockermigrate backup --out backup.tar.gz --containers adguardhome --verify

# Restore
./dockermigrate restore --in backup.tar.gz --bind-root / --pull-missing-images
```

## Notes

- Wenn ein Ziel-Container schon existiert, wird er beim Restore ersetzt (`docker rm -f`).
- Bei Backups mit Bind-Mounts ist `--bind-root` beim Restore Pflicht.
- Für systemnahe Zielpfade (z. B. `/data`, `/opt`, `/var/lib/...`) ist Restore häufig nur mit erhöhten Rechten möglich:

```bash
sudo ./dockermigrate restore --in backup.tar.gz --bind-root /
```

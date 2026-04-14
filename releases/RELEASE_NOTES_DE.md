# Docker Migration Tool v0.14.0

`dockermigrate` ist ein CLI-Tool, um Docker-Container inkl. Daten sauber zwischen Hosts zu migrieren.

## Highlights

- GUI-Lokalisierungsupdate:
  - integrierte Web-Oberfläche jetzt konsistent mit englischen Labels/Texten
  - Sicherheitsbestätigung und Statusmeldungen übersetzt
- GUI-UX konsistenter gemacht:
  - einheitliche Bezeichnungen bei Container-/Backup-Auswahl
  - Ordnerdialog sprachlich bereinigt
  - klare Aktionslabels für Backup/Restore/Verify
- Doku-Refresh:
  - README und lokales README an aktuelles GUI-Verhalten angepasst
  - deutscher Abschnitt sprachlich vereinheitlicht
- Vollständiger Release-Rebuild für alle Artefakte + neue Checksums

## Artefakte

- `dockermigrate-linux-amd64`
- `dockermigrate-darwin-amd64`
- `dockermigrate-darwin-arm64`
- `dockermigrate-windows-amd64.exe`
- `SHA256SUMS`

## Quick Start

```bash
# Backup
./dockermigrate backup --out backup.tar.gz --containers adguardhome --verify

# Restore
./dockermigrate restore --in backup.tar.gz --bind-root / --pull-missing-images
```

## Reales Testbeispiel (adguardhome)

```bash
./dockermigrate backup \
  --out /home/user/projects/dockermigrate/adguardhome-backup-2026-03-05-1819.tar.gz \
  --containers adguardhome \
  --verify

./dockermigrate restore \
  --in /home/user/projects/dockermigrate/adguardhome-backup-2026-03-05-1819.tar.gz \
  --bind-root / \
  --pull-missing-images
```

## Hinweis

Wenn der Ziel-Container bereits existiert, wird er beim Restore ersetzt (`docker rm -f <name>`).
Wenn Bind-Mounts im Backup enthalten sind, ist `--bind-root` beim Restore Pflicht.
Für systemnahe Zielpfade (z. B. `/data`, `/opt`, `/var/lib/...`) ist Restore häufig nur mit erhöhten Rechten möglich:

```bash
sudo ./dockermigrate restore --in backup.tar.gz --bind-root /
```

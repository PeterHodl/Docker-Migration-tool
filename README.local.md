# Docker Migration Tool (`dockermigrate`) — Local README

Aktueller lokaler Stand: **v0.11.0**

Dieses README ist für die **lokale Entwicklung/Nutzung** gedacht.

## Zweck

`dockermigrate` unterstützt die Migration von Docker-Workloads zwischen Hosts:

- Container-Konfiguration (aus `docker inspect`)
- Named Volumes
- Bind-Mount-Daten (optional)
- Optionales Image-Archiv
- Restore mit Zielprofilen

## Voraussetzungen

- Docker CLI + laufender Docker Daemon
- Go (nur für lokalen Build)

## Lokaler Build

```bash
go build -o dockermigrate main.go
```

Release-Builds lokal:

```bash
bash scripts/build-release.sh
```

## CLI (aktueller Stand)

```text
dockermigrate backup   --out backup.tar.gz [--containers c1,c2 | --all | --label key=value]
                      [--include-binds] [--include-images] [--bind-exclude PATTERN ...] [--dry-run]
                      [--no-stop] [--verify]

dockermigrate restore  --in backup.tar.gz --bind-root /some/root [--dry-run] [--pull-missing-images]
                      [--target auto|linux-arm64|linux-amd64|mac-silicon|mac-intel|windows-amd64]

dockermigrate doctor   [--in backup.tar.gz]
                      [--target auto|linux-arm64|linux-amd64|mac-silicon|mac-intel|windows-amd64]

dockermigrate plan     [--containers c1,c2 | --all | --label key=value]
                      [--include-binds] [--bind-exclude PATTERN ...] [--json]

dockermigrate ls       [--all]
dockermigrate verify   --in backup.tar.gz
dockermigrate version
```

## Verhalten (wichtig)

### Backup

- **Standard:** ausgewählte laufende Container werden vor Backup gestoppt und bleiben gestoppt.
- Mit `--no-stop`: Container laufen während Backup weiter.
- **Standard:** Images werden **nicht** als Tar mitgesichert.
- Mit `--include-images`: `images/images.tar` wird ins Backup geschrieben.
- Mit `--verify`: Archiv wird nach Erstellung direkt geprüft.

### Restore

- Wenn Bind-Mounts im Backup enthalten sind, ist `--bind-root` erforderlich.
- Falls `images/images.tar` fehlt, wird `docker load` übersprungen.
- Mit `--pull-missing-images` werden fehlende Images nachgezogen.
- `--target` steuert das Zielprofil / die Zielplattform.

### Doctor

- Prüft Docker-Erreichbarkeit + Host/Zielprofil.
- Optional mit `--in`: Backup-Kompatibilitätscheck.
- Status:
  - `GREEN` = ok
  - `YELLOW` = Warnungen
  - `RED` = Blocker (Exit 1)

## Beispiele

Plan:
```bash
./dockermigrate plan --containers adguardhome --json
```

Backup (Standard: stoppt Container, kein Image-Archiv):
```bash
./dockermigrate backup --out backup.tar.gz --containers adguardhome --verify
```

Backup ohne Stop + mit Image-Archiv:
```bash
./dockermigrate backup --out backup.tar.gz --containers adguardhome --no-stop --include-images
```

Doctor:
```bash
./dockermigrate doctor --in backup.tar.gz --target auto
```

Restore Dry-Run:
```bash
./dockermigrate restore --in backup.tar.gz --bind-root / --target auto --dry-run
```

Restore real:
```bash
./dockermigrate restore --in backup.tar.gz --bind-root / --target auto --pull-missing-images
```

Verify separat:
```bash
./dockermigrate verify --in backup.tar.gz
```

## Release-Artefakte

- `dockermigrate-linux-amd64`
- `dockermigrate-linux-arm64`
- `dockermigrate` (lokales convenience binary)
- `dockermigrate-darwin-amd64`
- `dockermigrate-darwin-arm64`
- `dockermigrate-windows-amd64.exe`

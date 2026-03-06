# Docker Migration Tool (`dockermigrate`)

## English

A CLI tool to migrate Docker containers between hosts, including:

- Container configuration (`docker inspect`)
- Image references (optional image tar)
- Named volumes
- Bind mount data
- Custom network recreation

Designed for: **backup on host A** → **restore on host B**.

### Features

- Planning before execution (`plan`)
- Dry-run for backup/restore
- Archive verification (`backup --verify`, `verify --in ...`)
- Bind policy: restore requires `--bind-root` if bind mounts are in backup
- Bind deduplication (same host path archived once)
- Extended restore support:
  - Labels, User, Workdir
  - Network mode + multi-network reconnect
  - Extra hosts
  - CapAdd / CapDrop
- Optional image archive in backup (`--include-images`, default: off)
- Automatic pull for missing images on restore (`--pull-missing-images`)
- Target profile auto-detection + override (`--target ...`)
- Preflight diagnostics via `doctor`

### Requirements

- Docker CLI + running Docker daemon
- Go (only required to build locally)

### Build

```bash
go build -o dockermigrate main.go
```

Multi-OS build:

```bash
bash scripts/build-release.sh
```

Artifacts are written to `releases/`.

### CLI

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

### Commands

#### `ls`
```bash
./dockermigrate ls
./dockermigrate ls --all
```

#### `plan`
```bash
./dockermigrate plan --containers adguardhome
./dockermigrate plan --all --json
./dockermigrate plan --label com.migrate=true
```

#### `backup`
```bash
./dockermigrate backup --out backup.tar.gz --containers adguardhome
```

Useful options:
- `--dry-run`
- `--verify`
- Default behavior: running selected containers are stopped before backup and remain stopped
- `--no-stop` to skip stopping
- `--bind-exclude` (repeatable)

Example:
```bash
./dockermigrate backup \
  --out backup.tar.gz \
  --containers app,db \
  --no-stop \
  --bind-exclude "/tmp" --bind-exclude "*cache*" \
  --verify
```

#### `verify`
```bash
./dockermigrate verify --in backup.tar.gz
```

#### `restore`
```bash
./dockermigrate restore --in backup.tar.gz --bind-root /
```

Restore notes:
- Existing target containers are replaced (`docker rm -f <name>`)
- `--bind-root` is required when binds are present
- Bind path mirroring happens under `bind-root`
- `--pull-missing-images` pulls missing images from manifest references
- If `images/images.tar` is missing, restore automatically switches to pull mode
- `--target` chooses target platform profile (default: `auto`)
- For system-level paths (`/data`, `/opt`, `/var/lib/...`), elevated privileges are often required:
  - `sudo ./dockermigrate restore --in backup.tar.gz --bind-root /`

Example:
```bash
./dockermigrate restore \
  --in backup.tar.gz \
  --bind-root / \
  --target auto \
  --pull-missing-images
```

### Real test example (adguardhome)

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

### Archive structure

```text
backup.tar.gz
├── manifest.json
├── containers/
│   └── <container>.inspect.json
├── images/
│   └── images.tar
├── volumes/
│   └── <volume>.tar
└── binds/
    └── _<host_path_as_filename>.tar
```

### Security / Operations

- Tar extraction is hardened against path traversal
- Docker socket binds are excluded by default:
  - `/var/run/docker.sock`
  - `/run/docker.sock`
- For production workloads: always test with `plan` + `--dry-run` first

### macOS target notes

- `dockermigrate-darwin-amd64` runs on Intel Macs.
- On Apple Silicon (M1/M2/M3), use a native arm64 build (`darwin-arm64`) for best compatibility/performance.
- Docker Desktop on macOS runs containers in a VM; Linux host paths are not directly portable.
- Prefer a macOS path for restore, e.g.:
  - `--bind-root /Users/<user>/docker-restore`
- Ensure restore target paths are allowed in Docker Desktop file sharing settings.
- Some Linux networking modes/features (especially `host`) may behave differently on macOS.

### Release artifacts

- `dockermigrate-linux-amd64`
- `dockermigrate-darwin-amd64`
- `dockermigrate-darwin-arm64`
- `dockermigrate-windows-amd64.exe`

---

## Deutsch

Ein CLI-Tool zur Migration von Docker-Containern zwischen Hosts, inklusive:

- Container-Konfiguration (`docker inspect`)
- Image-Referenzen (optionales Image-Tar)
- Named Volumes
- Bind-Mount-Daten
- Rekonstruktion von Custom Networks

Zielbild: **Backup auf Host A** → **Restore auf Host B**.

### Funktionen

- Plan-Modus vor Ausführung (`plan`)
- Dry-Run für Backup/Restore
- Archiv-Validierung (`backup --verify`, `verify --in ...`)
- Bind-Policy: Restore benötigt `--bind-root`, wenn Bind-Mounts enthalten sind
- Dedup für Binds (gleicher Host-Pfad wird einmal archiviert)
- Erweiterter Restore:
  - Labels, User, Workdir
  - Network Mode + Multi-Network Reconnect
  - Extra Hosts
  - CapAdd / CapDrop
- Optionales Image-Archiv im Backup (`--include-images`, Standard: aus)
- Automatisches Nachziehen fehlender Images beim Restore (`--pull-missing-images`)
- Zielprofil-Autoerkennung + Override (`--target ...`)
- Vorab-Pr�fung mit `doctor`

### Voraussetzungen

- Docker CLI + laufender Docker-Daemon
- Go (nur fürs lokale Build)

### Build

```bash
go build -o dockermigrate main.go
```

Multi-OS Build:

```bash
bash scripts/build-release.sh
```

Artefakte liegen in `releases/`.

### CLI

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

### Befehle

#### `ls`
```bash
./dockermigrate ls
./dockermigrate ls --all
```

#### `plan`
```bash
./dockermigrate plan --containers adguardhome
./dockermigrate plan --all --json
./dockermigrate plan --label com.migrate=true
```

#### `backup`
```bash
./dockermigrate backup --out backup.tar.gz --containers adguardhome
```

Nützliche Optionen:
- `--dry-run`
- `--verify`
- Standard: laufende, selektierte Container werden vor Backup gestoppt und bleiben gestoppt
- `--no-stop` für Backup ohne Stop
- `--bind-exclude` (mehrfach nutzbar)

Beispiel:
```bash
./dockermigrate backup \
  --out backup.tar.gz \
  --containers app,db \
  --no-stop \
  --bind-exclude "/tmp" --bind-exclude "*cache*" \
  --verify
```

#### `verify`
```bash
./dockermigrate verify --in backup.tar.gz
```

#### `restore`
```bash
./dockermigrate restore --in backup.tar.gz --bind-root /
```

Restore-Hinweise:
- Existierende Ziel-Container werden ersetzt (`docker rm -f <name>`)
- `--bind-root` ist Pflicht, wenn Bind-Mounts enthalten sind
- Bind-Pfade werden unterhalb von `bind-root` gespiegelt
- `--pull-missing-images` zieht fehlende Images anhand der Manifest-Referenzen nach
- Fehlt `images/images.tar`, wechselt Restore automatisch in den Pull-Modus
- `--target` w�hlt das Ziel-Plattformprofil (Standard: `auto`)
- Für systemnahe Pfade (`/data`, `/opt`, `/var/lib/...`) sind oft erhöhte Rechte nötig:
  - `sudo ./dockermigrate restore --in backup.tar.gz --bind-root /`

Beispiel:
```bash
./dockermigrate restore \
  --in backup.tar.gz \
  --bind-root / \
  --target auto \
  --pull-missing-images
```

### Reales Testbeispiel (adguardhome)

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

### Archiv-Struktur

```text
backup.tar.gz
├── manifest.json
├── containers/
│   └── <container>.inspect.json
├── images/
│   └── images.tar
├── volumes/
│   └── <volume>.tar
└── binds/
    └── _<host_path_as_filename>.tar
```

### Sicherheit / Betrieb

- Tar-Extraction ist gegen Path Traversal abgesichert
- Docker-Socket-Binds werden standardmäßig ausgeschlossen:
  - `/var/run/docker.sock`
  - `/run/docker.sock`
- Für Produktion immer erst mit `plan` + `--dry-run` testen

### Hinweise für macOS als Zielsystem

- `dockermigrate-darwin-amd64` ist für Intel-Macs.
- Für Apple Silicon (M1/M2/M3) ist ein nativer arm64-Build (`darwin-arm64`) empfehlenswert.
- Docker Desktop auf macOS nutzt eine VM; Linux-Hostpfade sind nicht 1:1 übertragbar.
- Für Restore besser einen macOS-Pfad verwenden, z. B.:
  - `--bind-root /Users/<user>/docker-restore`
- Prüfen, dass Zielpfade in Docker Desktop als File Sharing freigegeben sind.
- Einige Linux-Netzwerkmodi/-Features (v. a. `host`) verhalten sich auf macOS anders.

### Release-Artefakte

- `dockermigrate-linux-amd64`
- `dockermigrate-darwin-amd64`
- `dockermigrate-darwin-arm64`
- `dockermigrate-windows-amd64.exe`

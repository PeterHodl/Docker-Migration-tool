# Docker Migration Tool (`dockermigrate`)

Ein CLI-Tool zum Migrieren von Docker-Containern zwischen Hosts – inkl.:

- Container-Konfiguration (`docker inspect`)
- Images
- Named Volumes
- Bind-Mount-Daten (optional/standardmäßig aktiv)
- Netzwerk-Rekonstruktion (Custom Networks)

> Entwickelt für: "Backup auf Host A" → "Restore auf Host B".

---

## Features

- **Plan-Modus** vor jedem Lauf (`plan`)
- **Dry-Run** für Backup/Restore
- **Verify** für Backup-Archive (`backup --verify`, `verify --in ...`)
- **Bind-Mount Policy**: Restore verlangt `--bind-root`, wenn Bind-Mounts im Backup enthalten sind
- **Dedup von Bind-Backups** (identische Host-Pfade werden nur einmal gesichert)
- **Restore inkl. zusätzlicher Container-Optionen**:
  - Labels
  - User
  - Workdir
  - Network Mode + Multi-Network Reconnect
  - Extra Hosts
  - CapAdd / CapDrop
- **Optionales Pull fehlender Images** beim Restore (`--pull-missing-images`)

---

## Voraussetzungen

- Docker CLI + laufender Docker Daemon
- Go (nur zum lokalen Kompilieren nötig)

---

## Build

### Lokales Build

```bash
go build -o dockermigrate main.go
```

### Multi-OS Build (Linux/macOS/Windows)

```bash
bash scripts/build-release.sh
```

Erzeugte Dateien liegen in `releases/`.

---

## CLI Übersicht

```text
dockermigrate backup   --out backup.tar.gz [--containers c1,c2 | --all | --label key=value]
                      [--include-binds] [--bind-exclude PATTERN ...] [--dry-run]
                      [--stop-first] [--stop-timeout 30] [--verify]

dockermigrate restore  --in backup.tar.gz --bind-root /some/root [--dry-run] [--pull-missing-images]

dockermigrate plan     [--containers c1,c2 | --all | --label key=value]
                      [--include-binds] [--bind-exclude PATTERN ...] [--json]

dockermigrate ls       [--all]
dockermigrate verify   --in backup.tar.gz
dockermigrate version
```

---

## Befehle im Detail

## `ls`
Zeigt Container mit Image + Anzahl Volume/Bind-Mounts.

```bash
./dockermigrate ls
./dockermigrate ls --all
```

## `plan`
Zeigt, was gesichert würde (Container, Images, Volumes, Binds).

```bash
./dockermigrate plan --containers adguardhome
./dockermigrate plan --all --json
./dockermigrate plan --label com.migrate=true
```

## `backup`
Erzeugt ein `.tar.gz`-Archiv.

```bash
./dockermigrate backup --out backup.tar.gz --containers adguardhome
```

### Nützliche Optionen

- `--dry-run` → nur Vorschau
- `--verify` → Archiv nach dem Schreiben validieren
- `--stop-first` → selektierte laufende Container vorher stoppen und danach wieder starten
- `--stop-timeout 30` → Timeout für `docker stop`
- `--bind-exclude` (repeatable) → Hostpfade/Pattern aus Binds ausschließen

Beispiel:

```bash
./dockermigrate backup \
  --out backup.tar.gz \
  --containers app,db \
  --stop-first --stop-timeout 20 \
  --bind-exclude "/tmp" --bind-exclude "*cache*" \
  --verify
```

## `verify`
Prüft ein existierendes Backup-Archiv auf Konsistenz (Manifest + referenzierte Dateien).

```bash
./dockermigrate verify --in backup.tar.gz
```

## `restore`
Stellt Container aus einem Backup wieder her.

```bash
./dockermigrate restore --in backup.tar.gz --bind-root /
```

### Wichtige Restore-Hinweise

- Wenn der Ziel-Containername schon existiert, wird er durch `docker rm -f <name>` ersetzt.
- Falls Binds im Backup enthalten sind, ist `--bind-root` Pflicht.
- Spiegelung der Bind-Pfade erfolgt unterhalb von `bind-root`.
  - Beispiel: Quelle `/data/app/config`, `--bind-root /` → Ziel `/data/app/config`
  - Beispiel: Quelle `/data/app/config`, `--bind-root /restore-root` → Ziel `/restore-root/data/app/config`
- Mit `--pull-missing-images` werden fehlende Images nach `docker load` automatisch gepullt.

Beispiel:

```bash
./dockermigrate restore \
  --in backup.tar.gz \
  --bind-root / \
  --pull-missing-images
```

---

## Archiv-Struktur

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

---

## Sicherheits-/Betriebshinweise

- Tar-Extraction ist gegen Path Traversal abgesichert.
- Standardmäßig werden Docker-Socket-Binds ausgeschlossen:
  - `/var/run/docker.sock`
  - `/run/docker.sock`
- Für produktive Workloads erst mit `plan` + `--dry-run` testen.

---

## Release Artefakte

Die Multi-OS Binaries werden unter `releases/` erzeugt:

- `dockermigrate-linux-amd64`
- `dockermigrate-darwin-amd64`
- `dockermigrate-windows-amd64.exe`

(Architektur/Targets sind im Build-Script anpassbar.)

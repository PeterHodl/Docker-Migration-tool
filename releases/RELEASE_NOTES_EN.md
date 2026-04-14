# Docker Migration Tool v0.14.0

`dockermigrate` is a CLI tool to migrate Docker containers (including data) between hosts safely and reproducibly.

## Highlights

- GUI localization update:
  - built-in web UI now uses English labels/messages consistently
  - translated safety confirmation and runtime status texts
- GUI UX consistency improvements:
  - unified wording for container/backup selection
  - folder picker labels cleaned up
  - standardized action labels for backup/restore/verify
- Documentation refresh:
  - README and local README aligned with current GUI behavior
  - German section wording polished for consistency
- Full release rebuild for all artifacts + new checksums

## Artifacts

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

## Real test example (adguardhome)

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

## Note

If a target container already exists, restore replaces it (`docker rm -f <name>`).
If bind mounts are present in the backup, `--bind-root` is required during restore.
For system-level target paths (e.g. `/data`, `/opt`, `/var/lib/...`), restore often requires elevated privileges:

```bash
sudo ./dockermigrate restore --in backup.tar.gz --bind-root /
```

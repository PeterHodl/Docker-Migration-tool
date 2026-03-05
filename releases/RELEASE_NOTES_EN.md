# Docker Migration Tool v0.8.0

`dockermigrate` is a CLI tool to migrate Docker containers (including data) between hosts safely and reproducibly.

## Highlights

- Backup/restore of:
  - Container configuration
  - Images
  - Named volumes
  - Bind mount data
- Plan mode + dry-run for safe execution
- Archive validation (`verify`, `backup --verify`)
- Extended restore support:
  - Labels, user, workdir
  - Network mode + multi-network reconnect
  - Extra hosts, cap-add, cap-drop
- Optional pull of missing images:
  - `--pull-missing-images`
- Network metadata persisted in backups:
  - Driver, subnet, gateway (IPAM)

## Artifacts

- `dockermigrate-linux-amd64`
- `dockermigrate-darwin-amd64`
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

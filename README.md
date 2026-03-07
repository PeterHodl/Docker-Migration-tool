# Docker Migration Tool (`dockermigrate`)

> Public distribution repository for prebuilt release binaries.

## Overview

`dockermigrate` migrates Docker workloads between hosts, including:

- Container runtime configuration
- Named volumes
- Bind mount data
- Network recreation
- Optional image handling during restore

Typical flow:

1. Create backup on source host
2. Transfer archive
3. Restore on target host

## Installation

Download the binary for your platform from the **Releases** section and place it on your host.

Included release artifacts:

- `dockermigrate-linux-amd64`
- `dockermigrate-linux-arm64`
- `dockermigrate-darwin-amd64`
- `dockermigrate-darwin-arm64`
- `dockermigrate-windows-amd64.exe`

## CLI commands

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

## Recommended workflow

- Run `plan` first
- Use `--dry-run` before production backup/restore
- Run `verify` (or `backup --verify`) for archive integrity checks
- Use `doctor` on target host before restore

## Notes

- Existing target containers with same name are replaced during restore.
- If bind mounts are included, `--bind-root` is required for restore.
- If image archive is unavailable, restore can pull missing images (`--pull-missing-images`).

## Support

For issues and bug reports, use this repository’s issue tracker.

## License

This project is distributed under a proprietary license. See `LICENSE`.

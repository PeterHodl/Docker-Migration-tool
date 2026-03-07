# Docker Migration Tool (`dockermigrate`) — Public Release README

This repository is intended for **release distribution** (prebuilt binaries).

## Download / Installation

Download the matching binary from the **Releases** section:

- `dockermigrate-linux-amd64`
- `dockermigrate-linux-arm64`
- `dockermigrate-darwin-amd64`
- `dockermigrate-darwin-arm64`
- `dockermigrate-windows-amd64.exe`

Optional checksum verification:

- `SHA256SUMS`

## What the tool does

`dockermigrate` helps migrate Docker workloads from one system to another:

- backups from source host
- restore on target host
- planning and dry-runs
- optional archive verification

## Command overview

```text
dockermigrate backup  --out backup.tar.gz [--containers c1,c2 | --all | --label key=value]
                     [--dry-run] [--no-stop] [--include-images] [--verify]

dockermigrate restore --in backup.tar.gz [--dry-run] [--pull-missing-images]
                     [--target auto|linux-arm64|linux-amd64|mac-silicon|mac-intel|windows-amd64]

dockermigrate doctor  [--in backup.tar.gz]
                     [--target auto|linux-arm64|linux-amd64|mac-silicon|mac-intel|windows-amd64]

dockermigrate plan    [--containers c1,c2 | --all | --label key=value] [--json]
dockermigrate ls      [--all]
dockermigrate verify  --in backup.tar.gz
dockermigrate version
```

## Short command descriptions

- `backup`: create migration archive
- `restore`: restore a migration archive to target system
- `doctor`: preflight checks for environment and target profile
- `plan`: preview what would be included
- `ls`: list containers
- `verify`: verify archive integrity/structure
- `version`: print tool version

## Typical workflow

1. Run `plan` / `backup --dry-run`
2. Create backup
3. Transfer archive
4. Run `doctor` on target
5. Run restore (`--dry-run` first)

## Notes

- By default, backup stops selected running containers and keeps them stopped.
- Use `--no-stop` to keep containers running during backup.
- Images are not archived by default; use `--include-images` if needed.

## License

See `LICENSE`.

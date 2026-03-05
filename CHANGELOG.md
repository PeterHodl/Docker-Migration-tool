# Changelog

## v0.9.0

- Changed backup default behavior:
  - selected running containers are stopped before backup
  - containers remain stopped after backup
- Removed `--stop-first` and `--stop-timeout`
- Added `--no-stop` to explicitly skip stopping containers

## v0.8.0

- Added optional image pull during restore:
  - `--pull-missing-images`
- Persist and reuse network metadata in manifest:
  - driver, subnet, gateway (IPAM)
- Improved custom network recreation using manifest metadata

## v0.7.0

- Recreate custom Docker networks during restore
- Reconnect containers to multiple networks (`docker network connect`)
- Improved network handling for non-default networks

## v0.6.0

- Added archive verification:
  - `backup --verify`
  - `verify --in <backup.tar.gz>`
- Deduplicated bind-mount backups/restores by source path

## v0.5.0

- Initial migration baseline
- Backup/restore for containers, images, named volumes
- Bind-mount backup/restore with `--bind-root` policy
- Plan, dry-run, and listing support

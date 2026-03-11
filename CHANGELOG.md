# Changelog

## v0.12.0

- Added install-free local web GUI via `dockermigrate gui`
  - built-in HTML UI in the same binary (no extra runtime)
  - backup / restore / verify actions from browser
- GUI network support:
  - `--listen 0.0.0.0:<port>` for LAN access
  - short port form: `dockermigrate gui 9080`
- GUI safety and UX improvements:
  - restore safety confirmation for `bind-root=/`
  - backup directory input and automatic backup filename generation from selected container (`<container>.tar.gz`)
  - backup selector for restore path (`*.tar.gz` in selected folder)
  - automatic container list load + dropdown selection
  - modernized dark UI theme
- Restore fix for entrypoint handling:
  - correctly maps JSON entrypoint arrays to Docker run (`--entrypoint <bin>` + trailing args)

## v0.11.0

- Backup workflow update:
  - default behavior stops selected running containers before backup and keeps them stopped
  - new flag `--no-stop` keeps containers running during backup
- Image handling update:
  - image archive is now optional by default (no `images/images.tar` unless requested)
  - new flag `--include-images` enables image archive creation in backup
- Verification improvements:
  - `backup --verify` available again
  - `verify --in <backup.tar.gz>` available again
  - verify accepts archives without `images/images.tar`
- Restore compatibility improvements:
  - restore works when backup has no `images/images.tar` (skips `docker load`)
  - optional `--pull-missing-images` can fetch required images from manifest refs
- Platform / diagnostics:
  - restore target profiles supported via `--target ...`
  - `doctor` command available for preflight checks (`--in`, `--target`)

## v0.10.0

- Added restore target profile support:
  - `--target auto|linux-arm64|linux-amd64|mac-silicon|mac-intel|windows-amd64`
  - platform-aware `docker run --platform ...` and pull behavior
- Added `doctor` command for preflight checks:
  - host/runtime + docker connectivity
  - optional backup compatibility checks (`--in`)
  - risk hints for host networking / privileged mode

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

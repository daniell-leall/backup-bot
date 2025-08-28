# Backup-Bot
Docker image based on Ubuntu 22.04 with rsync, cron, rclone, and timezone support for automated backups.

## Overview
This Docker image is based on Ubuntu 22.04 with pre-installed tools for automated backups.
It allows scheduling backups using `rsync` and `cron`, with optional cloud sync via `rclone` (not yet configured).

## Pre-installed Software
- `rsync`            - file synchronization and backup
- `cron`             - scheduled task management
- `rclone`           - cloud storage sync (future use, not configured)
- `ca-certificates`  - SSL support
- `tzdata`           - timezone configuration

## Quick Start: Docker Run
Run the container interactively or in detached mode with all necessary volumes, environment variables, and cron job:

docker run -d \
  --name backup-bot \                     # Local container name
  --restart unless-stopped \              # Automatically restart if stopped
  -v /mnt/HDD4TB-FILES:/mnt/HDD4TB-FILES:ro \   # Map source files (read-only)
  -v /mnt/HDD4TB-BACKUP:/mnt/HDD4TB-BACKUP \   # Map backup destination
  -v /opt/backup/logs:/var/log/backup \        # Map logs folder
  -e TZ=America/Sao_Paulo \                   # Set your timezone
  danielleal404/backup-bot:latest \           # Docker Hub image to pull
  bash -c "
    # Cron job at 05:00 AM
    echo '00 5 * * * root rsync -ah --info=progress2 --partial --append-verify --delete \
    /mnt/HDD4TB-FILES/ /mnt/HDD4TB-BACKUP/backup/ >> /var/log/backup/backup.log 2>&1' \
    > /etc/cron.d/backup-cron &&
    chmod 644 /etc/cron.d/backup-cron &&
    crontab /etc/cron.d/backup-cron &&
    cron -f
  "

# Usage: Docker Compose
# Easily run the backup container using Docker Compose

version: "3.9"

services:
  backup:
    image: danielleal404/backup-bot:latest
    container_name: backup-bot
    restart: unless-stopped

    # Map your source files (read-only)
    volumes:
      - /mnt/HDD4TB-FILES:/mnt/HDD4TB-FILES:ro

    # Map your backup destination
      - /mnt/HDD4TB-BACKUP:/mnt/HDD4TB-BACKUP

    # Map logs folder
      - /opt/backup/logs:/var/log/backup

    environment:
      # Set your timezone
      - TZ=America/Sao_Paulo

    tty: true

    command: >
      bash -c "
        # Cron job at 05:00 AM
        echo '00 5 * * * root rsync -ah --info=progress2 --partial --append-verify --delete \
        /mnt/HDD4TB-FILES/ /mnt/HDD4TB-BACKUP/backup/ >> /var/log/backup/backup.log 2>&1' \
        > /etc/cron.d/backup-cron &&
        chmod 644 /etc/cron.d/backup-cron &&
        crontab /etc/cron.d/backup-cron &&
        cron -f
      "



## Notes
- `rclone` is installed but not yet used in the backup job; future releases will add cloud backup support.
- Logs are saved in `/var/log/backup/backup.log`.
- Adjust volume paths and timezone (`TZ`) according to your system.
- The cron job is set to run daily at 05:00 AM; modify the schedule in `docker run` or `docker-compose` as needed.

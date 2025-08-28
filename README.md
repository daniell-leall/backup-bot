# backup-bot
Docker image based on Ubuntu 22.04 with rsync, cron, rclone, and timezone support for automated backups.
# Ubuntu Backup-Bot

This is a **Docker image based on Ubuntu 22.04** with pre-installed tools for automated backups.

## Pre-installed Software

- `rsync` – file synchronization and backup  
- `cron` – scheduled task management  
- `rclone` – cloud storage sync (not yet configured in this release)  
- `ca-certificates` – SSL support  
- `tzdata` – timezone configuration  

## Quick Start: Docker Run

Run the container interactively with all necessary volumes, environment, and cron command:

docker run -d \
  --name backup-bot \                     # Name your container
  --restart unless-stopped \              # Automatically restart if stopped
  -v /mnt/HDD4TB-FILES:/mnt/HDD4TB-FILES:ro \   # Map source files (read-only)
  -v /mnt/HDD4TB-BACKUP:/mnt/HDD4TB-BACKUP \   # Map backup destination
  -v /opt/backup/logs:/var/log/backup \        # Map logs folder
  -e TZ=America/Sao_Paulo \                   # Set your timezone
  danielleal404/backup-bot:latest \
  bash -c "
    # Cron job at 05:00 AM
    echo '00 5 * * * root rsync -ah --info=progress2 --partial --append-verify --delete \
    /mnt/HDD4TB-FILES/ /mnt/HDD4TB-BACKUP/backup/ >> /var/log/backup/backup.log 2>&1' \
    > /etc/cron.d/backup-cron &&
    chmod 644 /etc/cron.d/backup-cron &&
    crontab /etc/cron.d/backup-cron &&
    cron -f
  "

## Usage: Docker Compose

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

- `rclone` is installed but not yet used in the backup job. Future releases will add cloud backup support.  
- Logs are saved in `/var/log/backup/backup.log`.  
- Adjust volume paths and timezone (`TZ`) according to your system.  
- The cron job is set to run every day at 05:00 AM. You can modify the schedule in the `docker run` or `docker-compose` command.  

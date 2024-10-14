# PostgreSQL Backup and Standby Server Setup with WAL

This guide provides a step-by-step workflow for setting up a **PostgreSQL backup** and **standby server** using Write-Ahead Logging (WAL) with `pg_basebackup`. The following instructions cover the process of setting up archiving, taking base backups, restoring, and configuring a standby server.

## Prerequisites

Ensure the following prerequisites are met:
- **PostgreSQL 14** or newer installed on both the primary and standby servers.
- **Superuser (root) access** to manage PostgreSQL services.
- **Filesystem permissions** to create directories for backups and archive storage.

---

## Table of Contents

1. [Primary Server Setup](#primary-server-setup)
2. [Backup Process](#backup-process)
3. [Restore Process](#restore-process)
4. [Standby Server Configuration](#standby-server-configuration)
5. [Start and Stop PostgreSQL Service](#start-and-stop-postgresql-service)
6. [Conclusion](#conclusion)

---

## Primary Server Setup

1. **Create Archive and Backup Directories**  
   On the primary server, create directories for WAL archiving and base backups:
   
   ```bash
   sudo mkdir /mnt/server/archivedir
   sudo mkdir /mnt/pg_backup

## Set Permissions
Set the correct ownership and permissions for these directories:
- sudo chown -R postgres:postgres /mnt/server/archivedir
- sudo chmod -R 700 /mnt/server/archivedir
- sudo chown -R postgres:postgres /mnt/pg_backup
- sudo chmod -R 700 /mnt/pg_backup

## Edit postgresql.conf for WAL Archiving
Modify the PostgreSQL configuration file (postgresql.conf) to enable WAL archiving:
- sudo nano /var/lib/postgresql/14/main/postgresql.conf
### Add or modify the following lines:
    archive_mode = on
    archive_command = 'cp %p /mnt/server/archivedir/%f'
    wal_level = replica
    max_wal_senders = 3
    wal_keep_size = 64

## Reload PostgreSQL Configuration
Reload the configuration to apply the changes:
- sudo -u postgres pg_ctl reload -D /var/lib/postgresql/14/main


# Backup Process

## Start a Base Backup
Use pg_basebackup to create a compressed base backup of the primary server:
- sudo -u postgres pg_basebackup -D /mnt/pg_backup -Ft -z -P -X stream

####  -D: Target directory for the backup.
####  -Ft: Create a tar-format backup.
####  -z: Compress the backup.
####  -P: Show progress.
####  -X stream: Stream WAL during the backup.

## Start and Stop WAL Backup Manually
Initiate and complete the backup process:
- SELECT pg_start_backup('backup_label', false, false);
After the backup is complete, stop the backup:
- SELECT * FROM pg_stop_backup(false, true);


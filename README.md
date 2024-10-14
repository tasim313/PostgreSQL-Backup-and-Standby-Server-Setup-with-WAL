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

# Restore Process
If you need to restore the database from WAL logs, follow these steps:
Set the Restore Command
In the PostgreSQL configuration file (postgresql.conf), specify the restore command:
- restore_command = 'cp /mnt/server/archivedir/%f %p'
Touch Recovery Signal File
Create a recovery.signal file to indicate PostgreSQL should recover:
- sudo touch /var/lib/postgresql/14/main/recovery.signal
This tells PostgreSQL to enter recovery mode and apply archived WAL files.

# Standby Server Configuration
To configure a standby server for replication, follow these steps:
Prepare the Standby Server
Ensure that PostgreSQL is installed and that the data directory on the standby server is correctly set up:
- sudo chown -R postgres:postgres /var/lib/postgresql/14/main
- sudo chmod -R 700 /var/lib/postgresql/14/main
### Start PostgreSQL on Standby
Start the PostgreSQL service in standby mode with immediate recovery:
- sudo -u postgres /usr/lib/postgresql/14/bin/pg_ctl -D /var/lib/postgresql/14/main -l /var/log/postgresql/postgresql-14-main.log start -m immediate

### Stop PostgreSQL
You can stop PostgreSQL on the standby server when needed:
- sudo -u postgres /usr/lib/postgresql/14/bin/pg_ctl -D /var/lib/postgresql/14/main -l /var/log/postgresql/postgresql-14-main.log stop

### Start and Stop PostgreSQL Service
Starting PostgreSQL
To start PostgreSQL on the primary or standby server:
- sudo -u postgres /usr/lib/postgresql/14/bin/pg_ctl -D /var/lib/postgresql/14/main start

### Stopping PostgreSQL
To stop PostgreSQL:
- sudo -u postgres /usr/lib/postgresql/14/bin/pg_ctl -D /var/lib/postgresql/14/main stop

### Restarting PostgreSQL
To restart PostgreSQL:
- sudo systemctl restart postgresql

# User and Role Creation
For PostgreSQL access, create the necessary roles and set a superuser password if not already done:
Log in to PostgreSQL:
- sudo -u postgres psql

### Create a superuser role:
- CREATE ROLE postgres WITH LOGIN SUPERUSER PASSWORD 'root';
Alternatively, you can log in directly:
- psql -U root -d postgres

  
### Key Concepts Covered:
- **WAL Archiving**: Enables continuous archiving of transaction logs for recovery and replication.
- **Base Backup**: A full backup using `pg_basebackup`.
- **Restore Process**: Defines how to restore using archived WAL logs.
- **Standby Server**: Configuration for a replication server in standby mode.
- **Permissions and Security**: Ensures proper ownership and permissions for directories and files.

This README covers the main steps to configure and manage PostgreSQL backup and replication with WAL. Feel free to adjust the commands and paths based on your specific environment.

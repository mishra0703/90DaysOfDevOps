# Day 19 – Shell Scripting Project: Log Rotation, Backup & Crontab

### Task : Apply everything from Days 16–18 in real-world mini projects.

---

## Challenge Tasks

### Task 1: Log Rotation Script

Create `log_rotate.sh` that:

1. Takes a log directory as an argument (e.g., `/var/log/myapp`)
2. Compresses `.log` files older than 7 days using `gzip`
3. Deletes `.gz` files older than 30 days
4. Prints how many files were compressed and deleted
5. Exits with an error if the directory doesn't exist

```bash
#!/bin/bash



if [ ! -d "$1" ]; then
        echo "Log Directory ("$1") didn't exist"
        exit 1
fi


display_Usage() {
        echo "Usage : ./log_rotate.sh < A Log Directory Path >"
        exit 1
}


log_dir=$1
backup_dir=$2


compress_log_files() {
        compressed_count=0

        log_files=($(find "${log_dir}" -maxdepth 1 -name "*.log" -mtime +7))
        if [ ${#log_files[@]} -eq 0 ]; then
                echo "No log files older than 7 days found!"
        else
                for file in "${log_files[@]}"; do
                        filename=$(basename "$file")
                        gzip -kvc "$file" > "$backup_dir/$filename.gz"
                        ((compressed_count++))
                done
                echo "Total files compressed: $compressed_count"
        fi
}


delete_older_backups() {
        deleted_count=0

        old_backups=($(find "$backup_dir" -maxdepth 1 -name "*.gz" -mtime +7))

        if [ ${#old_backups[@]} -eq 0 ]; then
                echo "All Backups are Fresh"
        else
                for file in "${old_backups[@]}"; do
                        rm -vf $file
                        ((deleted_count++))
                done
                echo "Total files deleted: $deleted_count"
        fi
}




if [ $# -eq 0 ]; then
        display_Usage
fi

compress_log_files
delete_older_backups

```

---

### Task 2: Server Backup Script

Create `server-backup.sh` that:

1. Takes a source directory and backup destination as arguments
2. Creates a timestamped `.tar.gz` archive (e.g., `backup-2026-02-08.tar.gz`)
3. Verifies the archive was created successfully
4. Prints archive name and size
5. Deletes backups older than 14 days from the destination
6. Handles errors — exit if source doesn't exist

```bash
#!/bin/bash


display_usage() {
        echo "Usage : ./server_backup.sh <Source Dir> <Destination Dir>"
        exit 1
}



source_dir=$1
backup_dir=$2
timestamp=$(date '+%Y-%m-%d-%H-%M-%S')


create_archieve() {
        echo
        if [ -d "$source_dir" ]; then
                tar -czvf "$backup_dir/backup-$timestamp.tar.gz" "$source_dir"
        else
                echo "Directory does not exist"
        fi
        echo
}


verify_archieve() {
        echo

        if [ -f "$backup_dir/backup-$timestamp.tar.gz" ]; then
                echo "Archieve created successfully"
        else
                echo "Archieve not created sucessfully"
        fi
        echo
}


archieve_info() {
        echo
        echo "*****Zip Info*****"
        echo
        archieve_name="backup-$timestamp.tar.gz"
        archieve_size=$(du -sh "$backup_dir/$archieve_name" | awk '{print $1}')
        echo "Backup Name : $archieve_name"
        echo "Backup Size : $archieve_size"
        echo
}


old_backups() {
        echo

        old_backups=($( find "$backup_dir" -maxdepth 1 -name "*.tar.gz" -mmin +30 ))


        if [ ${#old_backups[@]} -eq 0 ]; then
                echo "All backups are new"
        else
                echo "***** Cleaning Old Backups *****"
                echo
                for file in "${old_backups[@]}"; do
                        rm -f $file
                        echo "$file has been deleted"
                done
        fi
        echo
}




if [ $# -eq 0 ]; then
        display_usage
fi


if [ ! -d "$source_dir" ]; then
        echo "Source directory doesn't exist"
        exit 1
fi


create_archieve
verify_archieve
archieve_info
old_backups
```

---

### Task 3: Crontab

1. Read: `crontab -l` — what's currently scheduled?
2. Understand cron syntax:
   ```bash
   * * * * *  command
   │ │ │ │ │
   │ │ │ │ └── Day of week (0-7)
   │ │ │ └──── Month (1-12)
   │ │ └────── Day of month (1-31)
   │ └──────── Hour (0-23)
   └────────── Minute (0-59)
   ```

3. Write cron entries (in your markdown, don't apply if unsure) for:
   - Run `log_rotate.sh` every day at 2 AM
   - Run `backup.sh` every Sunday at 3 AM

```bash
00 2 * * * ./log_rotate.sh /var/log/logs        #Everyday at 2 AM
00 3 * * 7 ./server-backup.sh data temp         #Every Sunday at 3 AM
```

---

### Task 4: Combine — Scheduled Maintenance Script

Create `maintenance.sh` that:

1. Calls your log rotation function
2. Calls your backup function
3. Logs all output to `/var/log/maintenance.log` with timestamps
4. Write the cron entry to run it daily at 1 AM


```bash
#!/bin/bash


log_file="/var/log/maintenance.log"
timestamp=$(date '+%Y-%m-%d-%H-%M-%S')

log_rotation() {
        ./log_rotate.sh /var/log/logs
}


server_backup() {
        ./server_backup.sh data temp
}



maintenance_log() {
        echo
        echo "***** Maintenance Started *****"
        echo "Time : $timestamp"
        echo
        echo

        echo "----- Log Rotation Started -----"
        echo
        log_rotation
        echo
        echo "------ Log Rotation Finished -----"
        echo
        echo

        echo "----- Server Backup Initialized -----"
        echo
        server_backup
        echo
        echo "----- Server Backup Finished -----"
        echo
        echo

        echo
        echo
        echo "***** Maintenance Finished *****"
}



maintenance_log | tee -a "$log_file"
```

```bash
00 1 * * * ./home/ubuntu/maintenance.sh         #Everyday at 1 AM
```

---

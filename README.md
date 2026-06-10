# Linux Cron Jobs — Complete Guide

A comprehensive guide on how to create, manage, and monitor cron jobs in Linux,
including shell script examples and email notification setup.

---

## Table of Contents

- [What is a Cron Job?](#what-is-a-cron-job)
- [Cron Syntax](#cron-syntax)
- [Managing Cron Jobs](#managing-cron-jobs)
- [Cron Job Examples](#cron-job-examples)
- [Shell Script Examples](#shell-script-examples)
- [Email Notifications Setup](#email-notifications-setup)
- [Cron Job with Email Notification Script](#cron-job-with-email-notification-script)
- [Cron Logs & Monitoring](#cron-logs--monitoring)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

---

## What is a Cron Job?

A **cron job** is a scheduled task in Linux/Unix that runs automatically at a
specified time or interval. It is managed by the **cron daemon** (`crond`), a
background service that checks the crontab (cron table) file for scheduled tasks
and executes them at the defined times.

Common use cases include:
- Automated backups
- Log rotation and cleanup
- System health monitoring
- Sending scheduled reports via email
- Running deployment or maintenance scripts

---

## Cron Syntax

Every cron job follows this syntax:

```
* * * * * /path/to/command
│ │ │ │ │
│ │ │ │ └── Day of the week  (0-7, where 0 and 7 = Sunday)
│ │ │ └──── Month            (1-12)
│ │ └────── Day of the month (1-31)
│ └──────── Hour             (0-23)
└────────── Minute           (0-59)
```

### Special Characters

| Character | Meaning | Example |
|-----------|---------|---------|
| `*` | Every unit | `* * * * *` — every minute |
| `,` | Multiple values | `0 9,17 * * *` — at 9am and 5pm |
| `-` | Range of values | `0 9-17 * * *` — every hour from 9am to 5pm |
| `/` | Step values | `*/15 * * * *` — every 15 minutes |

### Special Shortcuts

| Shortcut | Equivalent | Description |
|----------|------------|-------------|
| `@reboot` | — | Run once at system startup |
| `@hourly` | `0 * * * *` | Run every hour |
| `@daily` | `0 0 * * *` | Run once a day at midnight |
| `@weekly` | `0 0 * * 0` | Run once a week on Sunday |
| `@monthly` | `0 0 1 * *` | Run once a month on the 1st |
| `@yearly` | `0 0 1 1 *` | Run once a year on Jan 1st |

---

## Managing Cron Jobs

### Open the Crontab Editor

```bash
crontab -e
```

> This opens the crontab file for the current user in the default text editor.
> To edit for a specific user (requires root): `sudo crontab -u username -e`

### List All Cron Jobs

```bash
# List cron jobs for current user
crontab -l

# List cron jobs for a specific user (requires root)
sudo crontab -u username -l
```

### Remove All Cron Jobs

```bash
crontab -r
```

> ⚠️ **Warning:** This removes ALL cron jobs for the current user. Use with caution.

### System-wide Cron Directories

Linux also supports system-wide cron jobs via these directories:

| Directory | Schedule |
|-----------|----------|
| `/etc/cron.hourly/` | Scripts run every hour |
| `/etc/cron.daily/` | Scripts run every day |
| `/etc/cron.weekly/` | Scripts run every week |
| `/etc/cron.monthly/` | Scripts run every month |
| `/etc/cron.d/` | Custom schedule files |

---

## Cron Job Examples

```bash
# Run a script every minute
* * * * * /home/user/scripts/check.sh

# Run a script every day at midnight
0 0 * * * /home/user/scripts/backup.sh

# Run a script every Monday at 8:00 AM
0 8 * * 1 /home/user/scripts/weekly-report.sh

# Run a script every 15 minutes
*/15 * * * * /home/user/scripts/monitor.sh

# Run a script on the 1st of every month at 6:00 AM
0 6 1 * * /home/user/scripts/monthly-cleanup.sh

# Run a script every weekday (Mon-Fri) at 9:00 AM
0 9 * * 1-5 /home/user/scripts/workday-task.sh

# Run a script at system reboot
@reboot /home/user/scripts/startup.sh

# Run a script every day at 11:30 PM
30 23 * * * /home/user/scripts/nightly-task.sh
```

---

## Shell Script Examples

### Example 1 — Basic Backup Script

```bash
#!/bin/bash

###############################################################
# Script Name : backup.sh
# Description : Backs up a specified directory to a backup
#               location with a timestamped folder name.
# Usage       : ./backup.sh
# Cron        : 0 2 * * * /home/user/scripts/backup.sh
###############################################################

# Variables
SOURCE_DIR="/var/www/html"                        # directory to back up
BACKUP_DIR="/backups"                             # where backups are stored
TIMESTAMP=$(date +"%Y-%m-%d_%H-%M-%S")           # timestamp for folder name
BACKUP_PATH="$BACKUP_DIR/backup_$TIMESTAMP"      # full backup path
LOG_FILE="/var/log/backup.log"                    # log file location

# Create backup directory if it does not exist
mkdir -p "$BACKUP_DIR"

# Perform the backup using rsync
echo "[$TIMESTAMP] Starting backup of $SOURCE_DIR..." | tee -a "$LOG_FILE"

if rsync -av --delete "$SOURCE_DIR" "$BACKUP_PATH" >> "$LOG_FILE" 2>&1; then
    echo "[$TIMESTAMP] Backup completed successfully: $BACKUP_PATH" | tee -a "$LOG_FILE"
else
    echo "[$TIMESTAMP] ERROR: Backup failed!" | tee -a "$LOG_FILE"
    exit 1
fi

# Delete backups older than 7 days
find "$BACKUP_DIR" -type d -name "backup_*" -mtime +7 -exec rm -rf {} + 2>/dev/null
echo "[$TIMESTAMP] Old backups cleaned up." | tee -a "$LOG_FILE"
```

---

### Example 2 — Disk Usage Monitor Script

```bash
#!/bin/bash

###############################################################
# Script Name : disk-monitor.sh
# Description : Monitors disk usage and logs a warning when
#               usage exceeds the defined threshold.
# Usage       : ./disk-monitor.sh
# Cron        : */30 * * * * /home/user/scripts/disk-monitor.sh
###############################################################

# Variables
THRESHOLD=80                                      # alert if disk usage exceeds 80%
LOG_FILE="/var/log/disk-monitor.log"             # log file location
TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")           # current timestamp

# Get current disk usage percentage for root partition
DISK_USAGE=$(df / | grep / | awk '{print $5}' | sed 's/%//')

echo "[$TIMESTAMP] Disk usage is at ${DISK_USAGE}%." | tee -a "$LOG_FILE"

# Check if disk usage exceeds threshold
if [ "$DISK_USAGE" -ge "$THRESHOLD" ]; then
    echo "[$TIMESTAMP] WARNING: Disk usage is at ${DISK_USAGE}% — threshold is ${THRESHOLD}%!" | tee -a "$LOG_FILE"
    exit 1
else
    echo "[$TIMESTAMP] Disk usage is within acceptable limits." | tee -a "$LOG_FILE"
fi
```

---

### Example 3 — Log Cleanup Script

```bash
#!/bin/bash

###############################################################
# Script Name : log-cleanup.sh
# Description : Deletes log files older than a defined number
#               of days from a specified directory.
# Usage       : ./log-cleanup.sh
# Cron        : 0 0 * * 0 /home/user/scripts/log-cleanup.sh
###############################################################

# Variables
LOG_DIR="/var/log/myapp"                          # directory containing logs
RETENTION_DAYS=30                                 # delete logs older than 30 days
TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")           # current timestamp
CLEANUP_LOG="/var/log/cleanup.log"               # cleanup log file

echo "[$TIMESTAMP] Starting log cleanup in $LOG_DIR..." | tee -a "$CLEANUP_LOG"

# Find and delete old log files
DELETED=$(find "$LOG_DIR" -type f -name "*.log" -mtime +$RETENTION_DAYS -print -delete 2>&1)

if [ -n "$DELETED" ]; then
    echo "[$TIMESTAMP] Deleted the following files:" | tee -a "$CLEANUP_LOG"
    echo "$DELETED" | tee -a "$CLEANUP_LOG"
else
    echo "[$TIMESTAMP] No log files older than $RETENTION_DAYS days found." | tee -a "$CLEANUP_LOG"
fi

echo "[$TIMESTAMP] Log cleanup complete." | tee -a "$CLEANUP_LOG"
```

---

## Email Notifications Setup

To receive email notifications from cron jobs, you need a mail transfer agent (MTA)
installed on your system.

### Step 1 — Install a Mail Utility

```bash
# On Ubuntu/Debian
sudo apt update
sudo apt install mailutils -y

# On CentOS/RHEL
sudo yum install mailx -y
```

### Step 2 — Install and Configure Postfix (SMTP)

```bash
# Install Postfix
sudo apt install postfix -y
```

During installation, select **"Internet Site"** and enter your domain name.

### Step 3 — Configure Postfix for Gmail SMTP Relay

Edit the Postfix configuration file:

```bash
sudo nano /etc/postfix/main.cf
```

Add or update the following lines:

```ini
relayhost = [smtp.gmail.com]:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
```

### Step 4 — Add Gmail Credentials

```bash
sudo nano /etc/postfix/sasl_passwd
```

Add this line with your Gmail credentials:

```
[smtp.gmail.com]:587 your-email@gmail.com:your-app-password
```

> ⚠️ **Important:** Use a **Gmail App Password**, not your regular Gmail password.
> To generate one: Google Account → Security → 2-Step Verification → App Passwords.

Secure and apply the credentials:

```bash
sudo postmap /etc/postfix/sasl_passwd          # hash the credentials file
sudo chmod 600 /etc/postfix/sasl_passwd        # restrict file permissions
sudo systemctl restart postfix                 # restart Postfix to apply changes
```

### Step 5 — Test Email Sending

```bash
echo "Test email from Linux cron job server" | mail -s "Test Email" your-email@gmail.com
```

If the email is received, your mail setup is working correctly.

### Step 6 — Set Cron Email Recipient

Add the `MAILTO` variable at the top of your crontab to specify who receives output:

```bash
crontab -e
```

```bash
# Send all cron output to this email address
MAILTO="your-email@gmail.com"

# Your cron jobs below
0 2 * * * /home/user/scripts/backup.sh
```

> Setting `MAILTO=""` disables all email notifications from cron.

---

## Cron Job with Email Notification Script

This is a complete, production-ready script that runs a task and sends an email
notification on both success and failure:

```bash
#!/bin/bash

###############################################################
# Script Name : backup-with-notify.sh
# Description : Performs a directory backup and sends an email
#               notification reporting success or failure.
#
# Usage       : ./backup-with-notify.sh
# Cron        : 0 2 * * * /home/user/scripts/backup-with-notify.sh
#
# Requirements: mailutils and postfix must be installed and
#               configured before running this script.
###############################################################

# ─── Configuration ────────────────────────────────────────────
SOURCE_DIR="/var/www/html"                         # directory to back up
BACKUP_DIR="/backups"                              # backup destination
TIMESTAMP=$(date +"%Y-%m-%d_%H-%M-%S")            # timestamp for naming
BACKUP_PATH="$BACKUP_DIR/backup_$TIMESTAMP"       # full backup path
LOG_FILE="/var/log/backup-notify.log"             # log file
EMAIL="your-email@gmail.com"                       # notification recipient
HOSTNAME=$(hostname)                               # server hostname
# ──────────────────────────────────────────────────────────────

# Create backup directory if it does not exist
mkdir -p "$BACKUP_DIR"

echo "[$TIMESTAMP] Backup started on $HOSTNAME" | tee -a "$LOG_FILE"
echo "[$TIMESTAMP] Source: $SOURCE_DIR → Destination: $BACKUP_PATH" | tee -a "$LOG_FILE"

# ─── Perform Backup ───────────────────────────────────────────
if rsync -av --delete "$SOURCE_DIR" "$BACKUP_PATH" >> "$LOG_FILE" 2>&1; then

    # ── Success ──────────────────────────────────────────────
    STATUS="SUCCESS"
    MESSAGE="✅ Backup completed successfully on $HOSTNAME at $TIMESTAMP.

Details:
  - Source      : $SOURCE_DIR
  - Destination : $BACKUP_PATH
  - Log File    : $LOG_FILE
  - Server      : $HOSTNAME

No action required."

    echo "[$TIMESTAMP] Backup completed successfully." | tee -a "$LOG_FILE"

else

    # ── Failure ───────────────────────────────────────────────
    STATUS="FAILED"
    MESSAGE="❌ Backup FAILED on $HOSTNAME at $TIMESTAMP.

Details:
  - Source      : $SOURCE_DIR
  - Destination : $BACKUP_PATH
  - Log File    : $LOG_FILE
  - Server      : $HOSTNAME

Please check the log file immediately and investigate the cause of failure."

    echo "[$TIMESTAMP] ERROR: Backup failed!" | tee -a "$LOG_FILE"

fi

# ─── Send Email Notification ──────────────────────────────────
echo "$MESSAGE" | mail -s "[$STATUS] Backup Report — $HOSTNAME — $TIMESTAMP" "$EMAIL"

# check if email was sent successfully
if [ $? -eq 0 ]; then
    echo "[$TIMESTAMP] Notification email sent to $EMAIL." | tee -a "$LOG_FILE"
else
    echo "[$TIMESTAMP] WARNING: Failed to send notification email." | tee -a "$LOG_FILE"
fi

# ─── Cleanup Old Backups ──────────────────────────────────────
# delete backups older than 7 days to free up disk space
find "$BACKUP_DIR" -type d -name "backup_*" -mtime +7 -exec rm -rf {} + 2>/dev/null
echo "[$TIMESTAMP] Backups older than 7 days removed." | tee -a "$LOG_FILE"

echo "[$TIMESTAMP] Script finished." | tee -a "$LOG_FILE"
```

### Schedule This Script in Crontab

```bash
crontab -e
```

```bash
MAILTO="your-email@gmail.com"

# Run backup with email notification every day at 2:00 AM
0 2 * * * /home/user/scripts/backup-with-notify.sh
```

### Email You Will Receive on Success

```
Subject: [SUCCESS] Backup Report — myserver — 2026-06-10_02-00-01

✅ Backup completed successfully on myserver at 2026-06-10_02-00-01.

Details:
  - Source      : /var/www/html
  - Destination : /backups/backup_2026-06-10_02-00-01
  - Log File    : /var/log/backup-notify.log
  - Server      : myserver

No action required.
```

### Email You Will Receive on Failure

```
Subject: [FAILED] Backup Report — myserver — 2026-06-10_02-00-01

❌ Backup FAILED on myserver at 2026-06-10_02-00-01.

Details:
  - Source      : /var/www/html
  - Destination : /backups/backup_2026-06-10_02-00-01
  - Log File    : /var/log/backup-notify.log
  - Server      : myserver

Please check the log file immediately and investigate the cause of failure.
```

---

## Cron Logs & Monitoring

### View Cron Logs

```bash
# Ubuntu/Debian
grep CRON /var/log/syslog

# CentOS/RHEL
sudo tail -f /var/log/cron

# Follow live cron activity
sudo tail -f /var/log/syslog | grep CRON
```

### Check if Cron Daemon is Running

```bash
sudo systemctl status cron        # Ubuntu/Debian
sudo systemctl status crond       # CentOS/RHEL
```

### Start / Restart / Stop Cron

```bash
sudo systemctl start cron         # start
sudo systemctl restart cron       # restart
sudo systemctl stop cron          # stop
sudo systemctl enable cron        # enable on boot
```

### Redirect Cron Output to a Log File

```bash
# Redirect both stdout and stderr to a log file
0 2 * * * /home/user/scripts/backup.sh >> /var/log/backup.log 2>&1

# Suppress all output (silent cron job)
0 2 * * * /home/user/scripts/backup.sh > /dev/null 2>&1
```

---

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| Cron job not running | Wrong file path | Always use absolute paths in crontab |
| Cron job not running | Script not executable | Run `chmod +x /path/to/script.sh` |
| Cron job not running | Cron daemon not active | Run `sudo systemctl start cron` |
| No email received | MAILTO not set | Add `MAILTO="you@email.com"` to crontab |
| No email received | Postfix not configured | Run `sudo systemctl status postfix` |
| Permission denied | Wrong script ownership | Run `chown user:user /path/to/script.sh` |
| Environment variable issues | Cron has minimal environment | Define variables explicitly inside the script |
| Script runs manually but not via cron | PATH not set | Add `PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin` to crontab |

### Common Fix — Always Use Absolute Paths

```bash
# ❌ Wrong — relative paths fail in cron
* * * * * ./backup.sh

# ✅ Correct — always use full absolute paths
* * * * * /home/user/scripts/backup.sh
```

### Common Fix — Set PATH in Crontab

```bash
crontab -e
```

```bash
# Add PATH at the top of your crontab
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

MAILTO="your-email@gmail.com"

0 2 * * * /home/user/scripts/backup.sh
```

---

## Best Practices

- **Always use absolute paths** for commands and scripts in cron jobs
- **Always redirect output** to a log file using `>> /path/to/logfile.log 2>&1`
- **Test your script manually** before scheduling it with cron
- **Use the `MAILTO` variable** to receive output and error notifications by email
- **Add comments** to your crontab entries to describe what each job does
- **Avoid overlapping jobs** — use lock files if a job may run longer than its interval
- **Use a cron expression validator** such as [crontab.guru](https://crontab.guru) to verify your schedule
- **Grant execute permission** to all scripts: `chmod 755 script.sh`
- **Log everything** — always write timestamps and status messages to a log file
- **Never store credentials** in scripts — use environment variables or secret managers instead

---

## Quick Reference Card

```bash
# Open crontab editor
crontab -e

# List cron jobs
crontab -l

# Remove all cron jobs
crontab -r

# View cron logs (Ubuntu)
grep CRON /var/log/syslog

# Test email sending
echo "test" | mail -s "Subject" you@email.com

# Check cron daemon status
sudo systemctl status cron

# Common cron schedules
* * * * *       # every minute
0 * * * *       # every hour
0 0 * * *       # every day at midnight
0 0 * * 0       # every Sunday at midnight
0 0 1 * *       # first day of every month
@reboot         # at system startup
```

---

*For more information, refer to the Linux man page:* `man 5 crontab`

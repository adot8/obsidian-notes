Cron jobs can also be set to run one time (such as on boot). They are typically used for administrative tasks such as running backups, cleaning up directories, etc. The `crontab` command can create a cron file, which will be run by the cron daemon on the schedule specified. When created, the cron file will be created in `/var/spool/cron` for the specific user that creates it. Each entry in the crontab file requires six items in the following order: minutes, hours, days, months, weeks, commands. For example, the entry `0 */12 * * * /home/admin/backup.sh` would run every 12 hours.

The root crontab is almost always only editable by the root user or a user with full sudo privileges; however, it can still be abused. You may find a world-writable script that runs as root and, even if you cannot read the crontab to know the exact schedule, you may be able to ascertain how often it runs (i.e., a backup script that creates a `.tar.gz` file every 12 hours). In this case, you can append a command onto the end of the script (such as a reverse shell one-liner), and it will execute the next time the cron job runs.

```bash
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```
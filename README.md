# Joomla bash backup script

This script backups all Joomla websites found below a specified directory.  
It will create a tar gzip (.tgz) or zip (.zip) file for each website and store it in a specified backup directory.  
Optionally, the script will upload the backup of each website to a Hetzner Storagebox with the rclone program.

The joomlabackups script uses the jfunctions script to retrieve information of the found Joomla websites.  
The jfunctions script reads the configuration.php and other files to determine the database credentials, website name and Joomla version.

## Workflow

The script does the following:  
1. Search all folders below the startdir folder to see if there is a configuration.php file.
2. Extract database credentials, website name and Joomla version.
3. Create a database dump of the joomla database.
4. Create a backup of the website folder (including the database dump) and store that in the backup directory.
5. If specified (-u option), upload the backup to a Hetzner Storagebox.
6. If specified (-t option) it adds a date/time stamp to the backup file name.
7. Remove database dump after the backup is created.
8. If specified (-d option) delete the local backup file after it is uploaded to the Hetzner Storagebox.

Syntax: `joomlabackups [-z] [-t] [-u] [-s] [-d] [-h]`

All options are optional.  
-z: create a .zip backup file instead of a .tgz file
-t: add a date/time stamp to the backup file name  
-u: upload the backup to a Hetzner Storagebox  
-s: silent, do not display any messages, only errors will be displayed  
-d: delete the local backup file after it is uploaded to the Hetzner Storagebox  
-h: display help  

The -s option is useful if you want to run the script as a cron job.

## Requirements

- Rclone installed and configured on your server to upload files to a Hetzner Storagebox.  
See also: <a href="https://rclone.org/" target="_blank">RClone website</a> and <a href="https://docs.hetzner.com/robot/storage-box/access/access-ssh-rsync-borg#rclone" target="_blank">Hetzner Storagebox with RClone</a>.
- Zip installed if you want to create .zip backups instead of .tgz backups.

## Installation

1. Make sure rclone is installed and configured on your server to upload files to a Hetzner Storagebox.
2. Download this repository.
3. Copy the scripts joomlabackups and jfunctions to a directory in your PATH, for example to /usr/local/bin.
4. Make the scripts executable with `chmod +x /usr/local/bin/joomlabackups /usr/local/bin/jfunctions`
5. Optionally change the variables in the joomlabackups script to match your server setup:  
STARTDIR: the directory where the script starts searching for Joomla sites  
STOREPATH: the directory where the backups are stored  
RCLONEDEST: the name of the rclone destination to upload the backups to (see ~/.config/rclone/rclone.conf)  
6. Optionally create a cronjob the run the joomlabackups script at regular intervals.  
An example cronjob that runs the script every day at 3:00 AM:  
`0 3 * * * /usr/local/bin/joomlabackups -u -d -s`

## Excluding sites from the backup

If you have a website in your startdir folder that you don't want to backup, you can do the following:  
Create a file `.nobackup` in the root of that website folder. This can be done with the command  
`touch /path/to/website/.nobackup`

When the backup scripts finds this file, it will skip the backup of that website.

## How are backups stored at the Hetzner Storagebox?

The backups are stored in a folder with the name of the website. A subfolder called `site` is created below that folder.  
Below the `site` folder, three folders are created: `day`, `week` and `month`.

All folders are created automatically by the script.

In the `day` folder, backups for each day except Sunday are stored.  
In the `week` folder, backups of the last 6 Sundays are stored.  
In the `month` folder, backups of the last 12 months are stored.

The day backup is run every day, Monday to Saturday.  
The week backup is run every Sunday.  
The month backup is run on every first day of each month.  

With this backupscheme, you keep backups for the last 6 weekdays, the last 6 Sundays and the last 12 months.  

## Backup filenames at the Hetzner Storagebox

All backups will be rotated automatically.

### Daily backups
Daily backups have the abbreviated dayname in the backup. For example, the Monday will be named as `/home/sitename/site/day/sitename.Mon.tgz`  
The next Monday, `sitename.Mon.tgz` will be overwritten with the new Monday backup.

### Weekly backups
The filenaming for weekly backups is different. The first week backup will be named as `/home/sitename/site/week/sitename.0.tgz`  
The next week, `sitename.0.tgz` will renamed to `sitename.1.tgz` and the new backup will be named `sitename.0.tgz`.  
The third week `sitename.1.tgz` will be renamed to `sitename.2.tgz` and `sitename.0.tgz` will be renamed to `sitename.1.tgz` and so on.  
This will go on until there are 6 weekly backups, number from 0 (newest week backup) to 5 (oldest week backup).

### Monthly backups
Monthly backups have the month number in the filename. The January backup will be named as `/home/sitename/site/month/sitename.01.tgz`  
The February backup will be named as `sitename.02.tgz` and so on until December.  
The next January, `sitename.01.tgz` will be overwritten with the new January backup.
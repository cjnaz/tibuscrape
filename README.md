# tibuscrape - Titanium Backup archiver

tibuscrape creates an archive of the versions of apps and their latest data files from your Titanium Backup.  

Each time that you update an app on your phone, TiBU normally creates a backup of the .apk file and the associated data files (.properties and .tar.gz).
Each successive TiBU run creates new data files up to your rolling "Max backup history" setting, and after that number of runs the older .apk version and it's data files are deleted forever.  

For example, I have "Max backup history" set to 4 and run the backup nightly.  This means that if I update an app I only have up to four days to find out that I'd rather stay on the older version.

This is where tibuscrape comes in:  **_tibuscrape monitors your local Dropbox or Google Drive copy of the TiBU "Remote location" backup directory and keeps an "archive" copy of every backed-up .apk version and the latest data files associated with that .apk version._** tibuscrape does not modify or delete any files in the TiBU backup directory.

## Setup and usage notes
- tibuscrape runs on Linux or Windows (tested on Python 2.7 and 3.6).  
- tibuscrape does not talk directly to the cloud service; rather, it relies on a local copy of the TiBU backup directory usually created by a local cloud sync agent or synced copies created by [rclonesync](https://github.com/cjnaz/rclonesync-V2) or [rclone](https://rclone.org/).
- Configure the `TIBU_PATH` and `ARCHIVE_PATH` constants in the script, or use the command line -T and -A switches.
- Manually create the target archive directory.
- The `--purge` switch may be used to automatically prune older versions from the archive.  If `--purge` is not specified then no files are deleted from the archive.  If `--purge` is specified, the default number of versions to keep is 3, but may be specified on the command line (i.e., `--purge 5` keeps 5 versions of each app).  You may also manually delete individual apps and their associated data files from the archive.
- The `--list` switch may be used to identify .properties/.tar.gz/.apk.gz file sets for installing back on the phone or manual deletion.
- To reinstall an archived app version and its data, copy the .apk.gz version and its associated .properties/.tar.gz data files to the backup directory on your phone using a file manager app (I use [Solid Explorer](https://play.google.com/store/apps/details?id=pl.solidexplorer2&hl=en_US)). Run Titanium Backup and drill down into the target app on the Backup/Restore tab.  Select to restore the App+Data for the older version.  You may have to uninstall the newer version first.
- Consider setting up a cron job to run tibuscrape some time after your scheduled Titanium Backup and Dropbox sync run.  Example cron with output redirect to a log file:

```
#        Minute  Hour    Day of Month   Month              Day of Week       Command    
#        (0-59)  (0-23)  (1-31)         (1-12 or Jan-Dec)  (0-6 or Sun-Sat)
           05      05      *              *                  *               (date; /<mypath to>/tibuscrape --verbose) >> /<mypath to>/runlog 2>&1
```
## CLI

(on Windows:  `> py tibuscrape -h`)

```
$ ./tibuscrape -T /mnt/raid1/share/public/DBox/Dropbox/TiBU/ -A /mnt/raid1/share/backups/TiBuScrapeArchive --purge -n -h
usage: tibuscrape [-h] [-T TIBU_PATH] [-A ARCHIVE_PATH] [-n] [-l]
                  [--purge [PURGE]] [-v] [-V]

Titanium Backup Scraper

optional arguments:
  -h, --help            show this help message and exit
  -T TIBU_PATH, --tibu-path TIBU_PATH
                        Path to the Titanium Backup directory.
  -A ARCHIVE_PATH, --archive-path ARCHIVE_PATH
                        Path to the Archive directory.
  -n, --dry-run         Copy/delete no files.
  -l, --list            Print content of the Archive directory and exit.
  --purge [PURGE]       Keep only n most recent versions in the archive - default 3.
  -v, --verbose         Print status and activity messages.
  -V, --version         Return version number and exit.
```

## Example run
```
$ ./tibuscrape -T /<path to>/Dropbox/TiBU/ -A /<path to>/TiBuScrapeArchive --verbose

***** Backup set latest datafiles *****
                            at.increase.wakeonlan__e741d5edc0d262a87c5e22ff704a4a3b -- 20190122-094122 -- Tue Jan 22 02:41:25 2019 -- Wake On Lan 1.4.9
                           co.happybits.marcopolo__3ccccc5c2c4f6eedca8e09a7658b0d40 -- 20190120-091641 -- Sun Jan 20 02:29:38 2019 -- Marco Polo 0.204.0
                           co.happybits.marcopolo__c4c9f4129606d2420753d021c47b71a5 -- 20190121-092642 -- Mon Jan 21 02:41:10 2019 -- Marco Polo 0.204.2
                           co.happybits.marcopolo__fe89a101f1849557312b148f98b30152 -- 20190122-092652 -- Tue Jan 22 02:34:34 2019 -- Marco Polo 0.205.0
                                com.amazon.kindle__d9082798a9deee56e5d392220ae2ddec -- 20190122-090005 -- Tue Jan 22 02:03:27 2019 -- Amazon Kindle 8.14.1.0
                com.amazon.mShop.android.shopping__2b8027975f3419888e6c0e5e32d58407 -- 20190121-090328 -- Mon Jan 21 02:03:48 2019 -- Amazon Shopping 18.1.0.100
                com.amazon.mShop.android.shopping__675a8939665613908ee908fadbbda99c -- 20190122-090327 -- Tue Jan 22 02:03:54 2019 -- Amazon Shopping 18.2.0.100
                                               com.android.providers.settings__none -- 20190122-093707 -- Tue Jan 22 02:37:08 2019 -- Settings storage 6.0.1
                                         com.android.providers.userdictionary__none -- 20190122-093818 -- Tue Jan 22 02:38:18 2019 -- User Dictionary 6.0.1
...

***** Archive set current datafiles *****
                            at.increase.wakeonlan__e741d5edc0d262a87c5e22ff704a4a3b -- 20190121-094801 -- Mon Jan 21 02:48:04 2019 -- Wake On Lan 1.4.9
                           co.happybits.marcopolo__2860042565e0e574e4e5c2a69201a63a -- 20190107-091338 -- Mon Jan  7 02:26:51 2019 -- Marco Polo 0.202.0
                           co.happybits.marcopolo__360897ef36bcc083db33e3989784bffa -- 20181231-091300 -- Mon Dec 31 02:16:45 2018 -- Marco Polo 0.193.0
                           co.happybits.marcopolo__3ccccc5c2c4f6eedca8e09a7658b0d40 -- 20190120-091641 -- Sun Jan 20 02:29:38 2019 -- Marco Polo 0.204.0
                           co.happybits.marcopolo__c4c9f4129606d2420753d021c47b71a5 -- 20190121-092642 -- Mon Jan 21 02:41:10 2019 -- Marco Polo 0.204.2
                           co.happybits.marcopolo__f627649ef6a09857b8cac23323403340 -- 20190114-091143 -- Mon Jan 14 02:24:48 2019 -- Marco Polo 0.203.0
                           co.happybits.marcopolo__f79465008ceaa147480058f995d5e951 -- 20190104-091403 -- Fri Jan  4 02:27:09 2019 -- Marco Polo 0.201.1
                                com.amazon.kindle__d9082798a9deee56e5d392220ae2ddec -- 20190121-090003 -- Mon Jan 21 02:03:28 2019 -- Amazon Kindle 8.14.1.0
                com.amazon.mShop.android.shopping__2b8027975f3419888e6c0e5e32d58407 -- 20190121-090328 -- Mon Jan 21 02:03:48 2019 -- Amazon Shopping 18.1.0.100
                                               com.android.providers.settings__none -- 20190121-094338 -- Mon Jan 21 02:43:38 2019 -- Settings storage 6.0.1
                                         com.android.providers.userdictionary__none -- 20190121-094456 -- Mon Jan 21 02:44:57 2019 -- User Dictionary 6.0.1
...

Archival transactions:
Saving new apk:          com.amazon.mShop.android.shopping-675a8939665613908ee908fadbbda99c.apk.gz
Saving new apk:          tunein.player-6c244e08ec2b21a52768d30f4944864c.apk.gz
Saving new apk:          com.intsig.camscanner-9a7bce8e67acd25e06385efeb3661fce.apk.gz
Saving new apk:          co.happybits.marcopolo-fe89a101f1849557312b148f98b30152.apk.gz
Saving new   datafiles:  Wake On Lan 1.4.9 -- /mnt/raid1/share/public/DBox/Dropbox/TiBU/at.increase.wakeonlan-20190122-094122
Removing old datafiles:  Wake On Lan 1.4.9 -- /mnt/raid1/share/backups/TiBuScrapeArchive/at.increase.wakeonlan-20190121-094801
Saving new   datafiles:  Marco Polo 0.205.0 -- /mnt/raid1/share/public/DBox/Dropbox/TiBU/co.happybits.marcopolo-20190122-092652
Saving new   datafiles:  Amazon Kindle 8.14.1.0 -- /mnt/raid1/share/public/DBox/Dropbox/TiBU/com.amazon.kindle-20190122-090005
Removing old datafiles:  Amazon Kindle 8.14.1.0 -- /mnt/raid1/share/backups/TiBuScrapeArchive/com.amazon.kindle-20190121-090003
Saving new   datafiles:  Amazon Shopping 18.2.0.100 -- /mnt/raid1/share/public/DBox/Dropbox/TiBU/com.amazon.mShop.android.shopping-20190122-090327
Saving new   datafiles:  Settings storage 6.0.1 -- /mnt/raid1/share/public/DBox/Dropbox/TiBU/com.android.providers.settings-20190122-093707
Removing old datafiles:  Settings storage 6.0.1 -- /mnt/raid1/share/backups/TiBuScrapeArchive/com.android.providers.settings-20190121-094338
Saving new   datafiles:  User Dictionary 6.0.1 -- /mnt/raid1/share/public/DBox/Dropbox/TiBU/com.android.providers.userdictionary-20190122-093818
Removing old datafiles:  User Dictionary 6.0.1 -- /mnt/raid1/share/backups/TiBuScrapeArchive/com.android.providers.userdictionary-20190121-094456
...

Archival operations tally:
     4   saved .apk.gz files
    60   saved datafiles
    58 deleted datafiles
     0 skipped new datafiles (due to missing backup .tar.gz file usually due to incomplete cloud sync)


***** Archive integrity checks *****
Updated Archive dataset:
                            at.increase.wakeonlan__e741d5edc0d262a87c5e22ff704a4a3b -- 20190122-094122 -- Tue Jan 22 02:41:25 2019 -- Wake On Lan 1.4.9
                           co.happybits.marcopolo__2860042565e0e574e4e5c2a69201a63a -- 20190107-091338 -- Mon Jan  7 02:26:51 2019 -- Marco Polo 0.202.0
                           co.happybits.marcopolo__360897ef36bcc083db33e3989784bffa -- 20181231-091300 -- Mon Dec 31 02:16:45 2018 -- Marco Polo 0.193.0
                           co.happybits.marcopolo__3ccccc5c2c4f6eedca8e09a7658b0d40 -- 20190120-091641 -- Sun Jan 20 02:29:38 2019 -- Marco Polo 0.204.0
                           co.happybits.marcopolo__c4c9f4129606d2420753d021c47b71a5 -- 20190121-092642 -- Mon Jan 21 02:41:10 2019 -- Marco Polo 0.204.2
                           co.happybits.marcopolo__f627649ef6a09857b8cac23323403340 -- 20190114-091143 -- Mon Jan 14 02:24:48 2019 -- Marco Polo 0.203.0
                           co.happybits.marcopolo__f79465008ceaa147480058f995d5e951 -- 20190104-091403 -- Fri Jan  4 02:27:09 2019 -- Marco Polo 0.201.1
                           co.happybits.marcopolo__fe89a101f1849557312b148f98b30152 -- 20190122-092652 -- Tue Jan 22 02:34:34 2019 -- Marco Polo 0.205.0
                                com.amazon.kindle__d9082798a9deee56e5d392220ae2ddec -- 20190122-090005 -- Tue Jan 22 02:03:27 2019 -- Amazon Kindle 8.14.1.0
                com.amazon.mShop.android.shopping__2b8027975f3419888e6c0e5e32d58407 -- 20190121-090328 -- Mon Jan 21 02:03:48 2019 -- Amazon Shopping 18.1.0.100
                com.amazon.mShop.android.shopping__675a8939665613908ee908fadbbda99c -- 20190122-090327 -- Tue Jan 22 02:03:54 2019 -- Amazon Shopping 18.2.0.100
                                               com.android.providers.settings__none -- 20190122-093707 -- Tue Jan 22 02:37:08 2019 -- Settings storage 6.0.1
                                         com.android.providers.userdictionary__none -- 20190122-093818 -- Tue Jan 22 02:38:18 2019 -- User Dictionary 6.0.1
...

Archive integrity checks tally (should all be 0, run with --verbose for more info):
     0 missing .apk.gz files (Archive has .properties that references a non-existing .apk.gz)
     0 missing .tar.gz files (Archive has .properties but no matching .tar.gz)
     0   extra .apk.gz files (Archive has no associated .properties/.tar.gz datafiles)
     0   extra .tar.gz files (Archive has .tar.gz but no matching .properties)
     0   extra datafiles     (More than one datafile set for a given .apk.gz)
```
## Known issues
- none

## Revision history
- 190129 v0.5 - Added --purge switch and improved log output
- 190122 v0.4 - Added archive integrity checks
- 190119 v0.3 - Added --list switch.
- 190105 v0.2 - Rewrite using .properties files internal data and not file datetime stamps.  Better logging.
- 190101 New

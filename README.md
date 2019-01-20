# tibuscrape - Titanium Backup archiver

tibuscrape creates an archive of the versions of apps and their latest data files from your Titanium Backup.  

Each time that you update an app on your phone, TiBU normally creates a backup of the .apk file and the associated data files (.properties and .tar.gz).
Each successive TiBU run creates new data files up to your rolling "Max backup history" setting, and after that number of runs the older .apk version and it's data files are deleted forever.  

For example, I have "Max backup history" set to 4 and run the backup nightly.  This means that if I update an app I only have up to four days to find out that I'd rather stay on the older version.

This is where tibuscrape comes in:  **_tibuscrape monitors your local Dropbox or Google Drive copy of the TiBU "Remote location" backup directory and keeps an "archive" copy of every backed-up .apk version and the latest data files associated with that .apk version._** tibuscrape does not modify or delete any files in the TiBU backup directory.

## Setup and usage notes
- tibuscrape runs on Linux or Windows (tested on Python 2.7 and 3.6).  
- tibuscrape does not talk directly to the cloud service; rather, it relies on a local copy of the TiBU backup directory usually created by a local cloud sync agent or synced copies created by [rclonesync](https://github.com/cjnaz/rclonesync-V2) or [rclone](https://rclone.org/).
- Configure the `TIBU_PATH` and `ARCHIVE_PATH` vars in the script, or use the command line -T and -A switches.
- Manually create the target archive directory.
- Periodically, manually delete older archived app versions and their data files.  Run `tibuscrape --list` switch to get a dump of the contents of the Archive directory contents.
- To reinstall an archived app and data version, identify the .apk.gz version and its associated .properties/.tar.gz data files using `tibuscrape --list` and copy the files to the backup directory on your phone using a file manager app (I use Solid Explorer). Run Titanium Backup and drill down into the target app on the Backup/Restore tab.  Select to restore the App+Data for the older version.  You may have to uninstall the newer version first.
- Consider setting up a cron job to run tibuscrape some time after your scheduled Titanium Backup and Dropbox sync run.  Example cron with output redirect to a log file:

```
#        Minute  Hour    Day of Month   Month              Day of Week       Command    
#        (0-59)  (0-23)  (1-31)         (1-12 or Jan-Dec)  (0-6 or Sun-Sat)
           05      05      *              *                  *               (date; /<mypath to>/tibuscrape --verbose) >> /<mypath to>/runlog 2>&1
```
## CLI

(on Windows:  `> py tibuscrape -h`)

```
$ ./tibuscrape -A /mnt/raid1/share/backups/TiBuScrapeArchive --h
usage: tibuscrape [-h] [-T TIBU_PATH] [-A ARCHIVE_PATH] [-n] [-l] [-v] [-V]

Titanium Backup Scraper

optional arguments:
  -h, --help            show this help message and exit
  -T TIBU_PATH, --tibu-path TIBU_PATH
                        Path to the Titanium Backup directory.
  -A ARCHIVE_PATH, --archive-path ARCHIVE_PATH
                        Path to the Archive directory.
  -n, --dry-run         Print status, but copy/delete no files.
  -l, --list            Print content of the Archive directory and exit.
  -v, --verbose         Print status and activity messages.
  -V, --version         Return version number and exit.

```

## Example run
```
$ ./tibuscrape -T /<path to>/Dropbox/TiBU/ -A /<path to>/TiBuScrapeArchive --verbose

Backup set latest datafiles:
                    at.increase.wakeonlan__e741d5edc0d262a87c5e22ff704a4a3b -- 20190105-092720 -- Sat Jan  5 02:27:23 2019 -- Wake On Lan 1.4.9
                   co.happybits.marcopolo__2860042565e0e574e4e5c2a69201a63a -- 20190105-091305 -- Sat Jan  5 02:26:15 2019 -- Marco Polo 0.202.0
                   co.happybits.marcopolo__f79465008ceaa147480058f995d5e951 -- 20190104-091403 -- Fri Jan  4 02:27:09 2019 -- Marco Polo 0.201.1
...
                             com.keramidas.virtual.BLUETOOTH_PAIRINGS__none -- 20190105-090018 -- Sat Jan  5 02:00:18 2019 -- Bluetooth Pairings
                                   com.keramidas.virtual.WIFI_AP_LIST__none -- 20190105-092723 -- Sat Jan  5 02:27:23 2019 -- Wi-Fi Access Points
                      com.mycelium.wallet__ad897c34a193b423e6c499a6c4e0b37d -- 20190105-092615 -- Sat Jan  5 02:26:20 2019 -- Mycelium Wallet 2.12.0.18
...
                       com.xmarks.android__dfab0f7c89faac96b7dde992be9b9137 -- 20181218-092224 -- Tue Dec 18 02:22:27 2018 -- Xmarks 1.0.16

Archive set latest datafiles:
                    at.increase.wakeonlan__e741d5edc0d262a87c5e22ff704a4a3b -- 20190103-091748 -- Thu Jan  3 02:17:50 2019 -- Wake On Lan 1.4.9
                   co.happybits.marcopolo__2860042565e0e574e4e5c2a69201a63a -- 20190105-091305 -- Sat Jan  5 02:26:15 2019 -- Marco Polo 0.202.0
                   co.happybits.marcopolo__360897ef36bcc083db33e3989784bffa -- 20181231-091300 -- Mon Dec 31 02:16:45 2018 -- Marco Polo 0.193.0
                   co.happybits.marcopolo__f79465008ceaa147480058f995d5e951 -- 20190104-091403 -- Fri Jan  4 02:27:09 2019 -- Marco Polo 0.201.1
...
                             com.keramidas.virtual.BLUETOOTH_PAIRINGS__none -- 20190105-090018 -- Sat Jan  5 02:00:18 2019 -- Bluetooth Pairings
                                   com.keramidas.virtual.WIFI_AP_LIST__none -- 20190105-092723 -- Sat Jan  5 02:27:23 2019 -- Wi-Fi Access Points
                      com.mycelium.wallet__867f22e8c397be29107ce21c596e7294 -- 20190104-092710 -- Fri Jan  4 02:27:10 2019 -- Mycelium Wallet 2.12.0.17
..
                       com.xmarks.android__dfab0f7c89faac96b7dde992be9b9137 -- Tue Dec 18 02:22:27 2018 -- Xmarks 1.0.16

Archival operations tally:
     0   saved .apk.gz files
     0   saved datafiles
     0 deleted datafiles
     0 skipped new datafiles (due to missing backup .tar.gz file usually due to incomplete cloud sync)

Archive integrity checks tally (should all be 0, run with --verbose for more info):
     0 missing .apk.gz files (Archive has .properties that references a non-existing .apk.gz)
     0 missing .tar.gz files (Archive has .properties but no matching .tar.gz)
     0   extra .apk.gz files (Archive has no associated .properties/.tar.gz datafiles)
     0   extra .tar.gz files (Archive has .tar.gz but no matching .properties)
     0   extra datafiles (more than one datafile set for a given .apk.gz)
```
## Known issues
- none

## Revision history
- 190120 v0.4 - Added archive integrity checks
- 190119 v0.3 - Added --list switch.
- 190105 v0.2 - Rewrite using .properties files internal data and not file datetime stamps.  Better logging.
- 190101 New

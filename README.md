# tibuscrape - Titanium Backup archiver

tibuscrape creates an archive of the versions of apps and their latest data files from your Titanium Backup.  

Each time that you update an app on your phone, Titanium Backup normally creates a backup of the .apk file and the associated data files (.properties and .tar.gz or .xml.gz).
Each successive Titanium Backup run creates new data files up to your rolling "Max backup history" setting, and after that number of runs the older .apk version and it's data files are deleted forever.  

For example, I have "Max backup history" set to 4 and run the backup nightly.  This means that if I update an app I only have up to four days to find out that I'd rather stay on the older version.

This is where tibuscrape comes in:  **_tibuscrape monitors your local Dropbox or Google Drive copy of the TitaniumBackup "Remote location" backup directory and keeps an "archive" copy of every backed-up .apk version and --the latest-- data files associated with that .apk version._** tibuscrape does not modify or delete any files in the TiBU backup directory.

**Definitions and key concpets:**
- Most apps have .tar.gz files, but some have .xml.gz files instead, such as the Messages backup.  An app data backup will have either a .tar.gz or .xml.gz, shorthand noted here as _.tar.gz/.xml.gz_.
- Most apps have a .apk.gz file, but some Android system backup items do not.  They are noted with an .apk.gz of _none_.
- For tibuscrape, an _AppDataSet_ is the collection of .properties, .tar.gz/.xml.gz, and their associated .apk.gz file.  In the example below, there are two AppDataSets in the archive for CamScanner (version for 5.9.0, and 5.9.1), and the datafiles for version 5.9.1 were updated on this run.  
- tibuscrape retains only the most recent AppDataSet info for _each version_ of an app, and up to the `--purge` count number of versions are retained in the archive for each app.

## Changes in the most recent release
- 191216 v0.7  Support AppDataSet versions of apps with "none" apk.gz files.  Prior releases kept only a single (latest) AppDataSet.  Note that since there are no app versions (because there is no .apk.gz), retaining a number of versions by tibuscrape is equivalent to retining a number of versions in Titanium Backup.  List AppDataSet sizes in MB.

## Setup and usage notes
- tibuscrape runs on Linux or Windows (tested on Python 2.7 and 3.7).  
- tibuscrape does not talk directly to the cloud service; rather, it relies on a local copy of the TitaniumBackup directory usually created by a local cloud sync agent or synced copies created by [rclonesync](https://github.com/cjnaz/rclonesync-V2) or [rclone](https://rclone.org/).
- Configure the `TIBU_PATH` and `ARCHIVE_PATH` constants in the script, or use the command line -T and -A switches.
- Manually create the target archive directory.
- The `--purge` switch may be used to automatically prune older versions from the archive.  If `--purge` is not specified then no files are deleted from the archive.  If `--purge` is specified, the default number of versions to keep is 3, but may be specified on the command line (i.e., `--purge 5` keeps 5 versions of each app).  `--purge 0` will delete everything from the archive (you be warned). `--purge` is available even in `--list` mode. You may also manually delete individual apps and their associated data files from the archive.
- The `--list` switch may be used to identify AppDataSets for installing back on the phone or for manual deletion.  Note that `--list` skips the backup-set --to-- archive-set comparison and archival transactions, and only lists the current content of the archive.
- To reinstall an AppDataSet copy the .apk.gz version and its associated .properties/.tar.gz/.xml.gz data files to the backup directory on your phone using a file manager app (I use [Solid Explorer](https://play.google.com/store/apps/details?id=pl.solidexplorer2&hl=en_US)). Run Titanium Backup and drill down into the target app on the Backup/Restore tab.  Select to restore the App+Data for the older version.  You may have to uninstall the current version first.
- Consider setting up a cron job to run tibuscrape some time after your scheduled Titanium Backup and Dropbox sync run.  Example cron with output redirect to a log file:

```
# Minute (0-59)
#      Hour (0-23)
#           Day of Month (1-31)
#                Month (1-12 or Jan-Dec)
#                     Day of Week (0-6 or Sun-Sat)
#                         Command    
  00   06   *    *    *   (echo; date; /<path_to>/tibuscrape -T /<path_to>/TitaniumBackup/ -A /<path_to>/TiBuScrapeArchive --verbose --purge 5) >> /<path_to>/tibuscrape/runlog 2>&1
```
## CLI

(on Windows:  `> py tibuscrape -h`)

```
$ ./tibuscrape -h
usage: tibuscrape [-h] [-T TIBU_PATH] [-A ARCHIVE_PATH] [-n] [-l]
                  [--purge [PURGE]] [-s] [-v] [-V]

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
  -s, --summary         Print an operations summary.
  -v, --verbose         Print status and activity messages.
  -V, --version         Return version number and exit.

```

## Example runs
```
$ ./tibuscrape -T /<path_to>/Dropbox/TiBU/ -A /<path_to>/TiBuScrapeArchive --summary

Archival transactions tally:
     3   saved .apk.gz files
    38   saved datafiles
    35 deleted datafiles
     2 skipped new datafiles (due to missing backup .tar.gz/.xml.gz file usually due to incomplete cloud sync)

Archive integrity checks tally (should all be 0, run with --verbose for more info):
     0 missing .apk.gz files         (Archive has .properties that references a non-existing .apk.gz)
     0 missing .tar.gz/.xml.gz files (Archive has .properties but no matching .tar.gz/.xml.gz)
     0   extra .apk.gz files         (Archive has no associated .properties/.tar.gz datafiles)
     0   extra .tar.gz files         (Archive has .tar.gz but no matching .properties)
     0   extra .xml.gz files         (Archive has .xml.gz but no matching .properties)
     0   extra datafiles             (More than one datafile set for a given .apk.gz)

Archive contains backups for  76  apps with  214  total versions

```
```
$ ./tibuscrape -T /<path_to>/Dropbox/TiBU/ -A /<path_to>/TiBuScrapeArchive --verbose --purge

***** Backup set latest datafiles *****
    Amazon Kindle 8.14.1.0                      --   70.7 MB -- Mon Jan 28 11:02:44 2019 -- d9082798a9deee56e5d392220ae2ddec -- com.amazon.kindle-20190128-175851
                (All lines actually list the AppDataSet size, as shown above.)
    Amazon Shopping 18.2.0.100                  -- Mon Jan 28 11:03:00 2019 -- 675a8939665613908ee908fadbbda99c -- com.amazon.mShop.android.shopping-20190128-180244
    Bluetooth Connect and Play 3.19             -- Mon Jan 28 11:03:01 2019 -- dc8d7a66d5c2271b2fda8774ba3d1543 -- com.cp2.start.and.play.music.player-20190128-180300
    Bluetooth Pairings                          -- Mon Jan 28 11:03:01 2019 -- none                             -- com.keramidas.virtual.BLUETOOTH_PAIRINGS-20190128-180301
    bVNC Pro v4.0.1                             -- Mon Jan 28 02:04:37 2019 -- 347ef1517f5ea7af5c0e5567ca79bc15 -- com.iiordanov.bVNC-20190128-090414
    bVNC Pro v4.0.3                             -- Mon Jan 28 11:03:25 2019 -- afcdaf4d8b4c670c24126ddc2c9b0edc -- com.iiordanov.bVNC-20190128-180301
    b\u00b7hyve 1.7.30                          -- Mon Jan 28 11:03:26 2019 -- 981f04bc7ae5b6b4ba64eaed4cdd1481 -- com.orbit.orbitsmarthome-20190128-180325
    Calendar 6.0.18-228718019-release           -- Mon Jan 28 11:03:29 2019 -- 236fd5c0321e3fbd97d74995b8bac03e -- com.google.android.calendar-20190128-180326
    CamScanner 5.9.0.20190116                   -- Sun Jan 27 02:09:16 2019 -- 9a7bce8e67acd25e06385efeb3661fce -- com.intsig.camscanner-20190127-090651
    CamScanner 5.9.1.20190126                   -- Mon Jan 28 11:05:51 2019 -- 3fdd290f93765b1703c36ba1393fa7eb -- com.intsig.camscanner-20190128-180329
...
    Google Play Books 5.0.5_RC04.227721962      -- Mon Jan 28 11:26:00 2019 -- 841e0b13dca884c33bcbec5e26cc628c -- com.google.android.apps.books-20190128-182010
...
    Plex 7.9.0.8439                             -- Mon Jan 28 11:35:04 2019 -- dd229309cb03775a42a45f92b7d57b72 -- com.plexapp.android-20190128-183422
    Podcast Republic 19.01.15R                  -- Sun Jan 27 02:37:17 2019 -- f92a636d822ea0d7fc89ce54f990d36d -- com.itunestoppodcastplayer.app-20190127-093706
    Podcast Republic 19.01.28R                  -- Mon Jan 28 11:35:15 2019 -- 28be82cec85691945f1abe4d12418486 -- com.itunestoppodcastplayer.app-20190128-183504
...
    xBrowserSync 1.4.0                          -- Mon Jan 28 11:40:50 2019 -- e93715bb65db886a6c5d16b948689d9a -- com.xBrowserSync.android-20190128-184045

***** Archive set preexisting datafiles *****
    Amazon Kindle 8.14.1.0                      -- Sun Jan 27 02:06:06 2019 -- d9082798a9deee56e5d392220ae2ddec -- com.amazon.kindle-20190127-090237
    Amazon Shopping 18.1.0.100                  -- Mon Jan 21 02:03:48 2019 -- 2b8027975f3419888e6c0e5e32d58407 -- com.amazon.mShop.android.shopping-20190121-090328
    Amazon Shopping 18.2.0.100                  -- Mon Jan 28 02:04:13 2019 -- 675a8939665613908ee908fadbbda99c -- com.amazon.mShop.android.shopping-20190128-090357
    Bluetooth Connect and Play 3.19             -- Mon Jan 28 02:04:14 2019 -- dc8d7a66d5c2271b2fda8774ba3d1543 -- com.cp2.start.and.play.music.player-20190128-090414
    Bluetooth Pairings                          -- Mon Jan 28 02:04:14 2019 -- none                             -- com.keramidas.virtual.BLUETOOTH_PAIRINGS-20190128-090414
    bVNC Pro v4.0.1                             -- Mon Jan 28 02:04:37 2019 -- 347ef1517f5ea7af5c0e5567ca79bc15 -- com.iiordanov.bVNC-20190128-090414
    b\u00b7hyve 1.7.30                          -- Mon Jan 28 02:04:38 2019 -- 981f04bc7ae5b6b4ba64eaed4cdd1481 -- com.orbit.orbitsmarthome-20190128-090437
    Calendar 6.0.12-224984167-release           -- Thu Jan 24 02:04:35 2019 -- 7eb1f865be1747d48cb08acb5722c124 -- com.google.android.calendar-20190124-090432
    Calendar 6.0.18-228718019-release           -- Mon Jan 28 02:04:41 2019 -- 236fd5c0321e3fbd97d74995b8bac03e -- com.google.android.calendar-20190128-090439
    CamScanner 5.9.0.20190116                   -- Sun Jan 27 02:09:16 2019 -- 9a7bce8e67acd25e06385efeb3661fce -- com.intsig.camscanner-20190127-090651
    CamScanner 5.9.1.20190126                   -- Mon Jan 28 02:07:34 2019 -- 3fdd290f93765b1703c36ba1393fa7eb -- com.intsig.camscanner-20190128-090442
...
    Google Play Books 5.0.5_RC04.227721962      -- Mon Jan 28 02:27:39 2019 -- 841e0b13dca884c33bcbec5e26cc628c -- com.google.android.apps.books-20190128-092147
...
    Messages (SMS & MMS)                        -- Sun Oct  6 10:52:22 2019 -- none                             --
    Plex 7.9.0.8439                             -- Sun Jan 27 02:37:06 2019 -- dd229309cb03775a42a45f92b7d57b72 -- com.plexapp.android-20190127-093625
    Podcast Republic 19.01.15R                  -- Sun Jan 27 02:37:17 2019 -- f92a636d822ea0d7fc89ce54f990d36d -- com.itunestoppodcastplayer.app-20190127-093706
    Podcast Republic 19.01.28R                  -- Mon Jan 28 02:37:01 2019 -- 28be82cec85691945f1abe4d12418486 -- com.itunestoppodcastplayer.app-20190128-093646
...
    xBrowserSync 1.4.0                          -- Mon Jan 28 02:42:39 2019 -- e93715bb65db886a6c5d16b948689d9a -- com.xBrowserSync.android-20190128-094234

***** Archival transactions *****
Saving new apk:          com.itunestoppodcastplayer.app-28be82cec85691945f1abe4d12418486.apk.gz
Saving new apk:          com.iiordanov.bVNC-afcdaf4d8b4c670c24126ddc2c9b0edc.apk.gz
Saving new   datafiles:  Amazon Kindle 8.14.1.0 -- /path_to/Dropbox/TiBU/com.amazon.kindle-20190128-175851
Removing old datafiles:  Amazon Kindle 8.14.1.0 -- /path_to/TiBuScrapeArchive/com.amazon.kindle-20190127-090237
Saving new   datafiles:  Amazon Shopping 18.2.0.100 -- /path_to/Dropbox/TiBU/com.amazon.mShop.android.shopping-20190128-180244
Removing old datafiles:  Amazon Shopping 18.2.0.100 -- /path_to/TiBuScrapeArchive/com.amazon.mShop.android.shopping-20190128-090357
Saving new   datafiles:  Bluetooth Connect and Play 3.19 -- /path_to/Dropbox/TiBU/com.cp2.start.and.play.music.player-20190128-180300
Removing old datafiles:  Bluetooth Connect and Play 3.19 -- /path_to/TiBuScrapeArchive/com.cp2.start.and.play.music.player-20190128-090414
Saving new   datafiles:  Bluetooth Pairings -- /path_to/Dropbox/TiBU/com.keramidas.virtual.BLUETOOTH_PAIRINGS-20190128-180301
Removing old datafiles:  Bluetooth Pairings -- /path_to/TiBuScrapeArchive/com.keramidas.virtual.BLUETOOTH_PAIRINGS-20190128-090414
Saving new   datafiles:  bVNC Pro v4.0.3 -- /path_to/Dropbox/TiBU/com.iiordanov.bVNC-20190128-180301
Saving new   datafiles:  b\u00b7hyve 1.7.30 -- /path_to/Dropbox/TiBU/com.orbit.orbitsmarthome-20190128-180325
Removing old datafiles:  b\u00b7hyve 1.7.30 -- /path_to/TiBuScrapeArchive/com.orbit.orbitsmarthome-20190128-090437
Saving new   datafiles:  Calendar 6.0.18-228718019-release -- /path_to/Dropbox/TiBU/com.google.android.calendar-20190128-180326
Removing old datafiles:  Calendar 6.0.18-228718019-release -- /path_to/TiBuScrapeArchive/com.google.android.calendar-20190128-090439
Saving new   datafiles:  CamScanner 5.9.1.20190126 -- /path_to/Dropbox/TiBU/com.intsig.camscanner-20190128-180329
Removing old datafiles:  CamScanner 5.9.1.20190126 -- /path_to/TiBuScrapeArchive/com.intsig.camscanner-20190128-090442
...
SKIPPING due to missing .tar.gz -- Google Play Books 5.0.5_RC04.227721962 -- /path_to/Dropbox/TiBU/com.google.android.apps.books-20190128-182010
...
Saving new   datafiles:  Plex 7.9.0.8439 -- /path_to/Dropbox/TiBU/com.plexapp.android-20190128-183422
Removing old datafiles:  Plex 7.9.0.8439 -- /path_to/TiBuScrapeArchive/com.plexapp.android-20190127-093625
Saving new   datafiles:  Podcast Republic 19.01.28R -- /path_to/Dropbox/TiBU/com.itunestoppodcastplayer.app-20190128-183504
Removing old datafiles:  Podcast Republic 19.01.28R -- /path_to/TiBuScrapeArchive/com.itunestoppodcastplayer.app-20190128-093646
...
Saving new   datafiles:  xBrowserSync 1.4.0 -- /path_to/Dropbox/TiBU/com.xBrowserSync.android-20190128-184045
Removing old datafiles:  xBrowserSync 1.4.0 -- /path_to/TiBuScrapeArchive/com.xBrowserSync.android-20190128-094234

Archival transactions tally:
     2   saved .apk.gz files
    59   saved datafiles
    58 deleted datafiles
     1 skipped new datafiles (due to missing backup .tar.gz file usually due to incomplete cloud sync)

***** Purging archive to <3> max versions *****

Purged  0  older apk.gz/.properties/.tar.gz sets from the Archive

***** Archive current content and integrity checks *****
    Amazon Kindle 8.14.1.0                      -- Mon Jan 28 11:02:44 2019 -- d9082798a9deee56e5d392220ae2ddec -- com.amazon.kindle-20190128-175851
    Amazon Shopping 18.1.0.100                  -- Mon Jan 21 02:03:48 2019 -- 2b8027975f3419888e6c0e5e32d58407 -- com.amazon.mShop.android.shopping-20190121-090328
    Amazon Shopping 18.2.0.100                  -- Mon Jan 28 11:03:00 2019 -- 675a8939665613908ee908fadbbda99c -- com.amazon.mShop.android.shopping-20190128-180244
    Bluetooth Connect and Play 3.19             -- Mon Jan 28 11:03:01 2019 -- dc8d7a66d5c2271b2fda8774ba3d1543 -- com.cp2.start.and.play.music.player-20190128-180300
    Bluetooth Pairings                          -- Mon Jan 28 11:03:01 2019 -- none                             -- com.keramidas.virtual.BLUETOOTH_PAIRINGS-20190128-180301
    bVNC Pro v4.0.1                             -- Mon Jan 28 02:04:37 2019 -- 347ef1517f5ea7af5c0e5567ca79bc15 -- com.iiordanov.bVNC-20190128-090414
    bVNC Pro v4.0.3                             -- Mon Jan 28 11:03:25 2019 -- afcdaf4d8b4c670c24126ddc2c9b0edc -- com.iiordanov.bVNC-20190128-180301
    b\u00b7hyve 1.7.30                          -- Mon Jan 28 11:03:26 2019 -- 981f04bc7ae5b6b4ba64eaed4cdd1481 -- com.orbit.orbitsmarthome-20190128-180325
    Calendar 6.0.12-224984167-release           -- Thu Jan 24 02:04:35 2019 -- 7eb1f865be1747d48cb08acb5722c124 -- com.google.android.calendar-20190124-090432
    Calendar 6.0.18-228718019-release           -- Mon Jan 28 11:03:29 2019 -- 236fd5c0321e3fbd97d74995b8bac03e -- com.google.android.calendar-20190128-180326
    CamScanner 5.9.0.20190116                   -- Sun Jan 27 02:09:16 2019 -- 9a7bce8e67acd25e06385efeb3661fce -- com.intsig.camscanner-20190127-090651
    CamScanner 5.9.1.20190126                   -- Mon Jan 28 11:05:51 2019 -- 3fdd290f93765b1703c36ba1393fa7eb -- com.intsig.camscanner-20190128-180329
...
    Google Play Books 5.0.5_RC04.227721962      -- Mon Jan 28 02:27:39 2019 -- 841e0b13dca884c33bcbec5e26cc628c -- com.google.android.apps.books-20190128-092147
...
    Messages (SMS & MMS)                        -- Sun Oct  6 10:52:22 2019 -- none                             -- com.keramidas.virtual.XML_MESSAGES-20191006-174511
    Plex 7.9.0.8439                             -- Mon Jan 28 11:35:04 2019 -- dd229309cb03775a42a45f92b7d57b72 -- com.plexapp.android-20190128-183422
    Podcast Republic 19.01.15R                  -- Sun Jan 27 02:37:17 2019 -- f92a636d822ea0d7fc89ce54f990d36d -- com.itunestoppodcastplayer.app-20190127-093706
    Podcast Republic 19.01.28R                  -- Mon Jan 28 11:35:15 2019 -- 28be82cec85691945f1abe4d12418486 -- com.itunestoppodcastplayer.app-20190128-183504
...
    xBrowserSync 1.4.0                          -- Mon Jan 28 11:40:50 2019 -- e93715bb65db886a6c5d16b948689d9a -- com.xBrowserSync.android-20190128-184045

Archive integrity checks tally (should all be 0, run with --verbose for more info):
     0 missing .apk.gz files         (Archive has .properties that references a non-existing .apk.gz)
     0 missing .tar.gz/.xml.gz files (Archive has .properties but no matching .tar.gz/.xml.gz)
     0   extra .apk.gz files         (Archive has no associated .properties/.tar.gz datafiles)
     0   extra .tar.gz files         (Archive has .tar.gz but no matching .properties)
     0   extra .xml.gz files         (Archive has .xml.gz but no matching .properties)
     0   extra datafiles             (More than one datafile set for a given .apk.gz)

Archive contains backups for  76  apps with  214  total versions
```

## Known issues
- none

## Revision history
- 191217 v0.7  Support AppDataSet versions of apps with "none" apk.gz files.  List AppDataSet sizes in MB.
- 191007 v0.6 - Support .xml.gz files, such as from MESSAGES backups (SMS/MMS).  Archive list is sorted with case-ignore.  Added --summary switch.
- 190128 v0.5 - Added --purge switch and improved log output
- 190122 v0.4 - Added archive integrity checks
- 190119 v0.3 - Added --list switch.
- 190105 v0.2 - Rewrite using .properties files internal data and not file datetime stamps.  Better logging.
- 190101 New

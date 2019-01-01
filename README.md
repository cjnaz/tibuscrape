# tibuscrape - Titanium Backup archiver

tibuscrape creates an archive of the versions of apps and their latest data files from your Titanium Backup.  

Each time that you update an app on your phone, TiBU normally creates a backup of the .apk file and the associated data files (.properties and .tar.gz).  Each successive TiBU run creates new data files up to your rolling "Max backup history" setting, and after that number of runs the older .apk version and it's data files are deleted forever.  

For example, I have "Max backup history" set to 4 and run the backup nightly.  This means that if I update an app I only have up to four days to find out that I'd rather stay on the older version.

This is where tibuscrape comes in:  **_tibuscrape monitors your local Dropbox or Google Drive copy of the TiBU "Remote location" backup directory and keeps an "archive" copy of every backed-up .apk version and the latest data files associated with that .apk version._** tibuscrape does not modify or delete any files in the TiBU backup directory.

## Setup and usage notes
- tibuscrape runs on Linux or Windows (tested on Python 2.7 and 3.6).  
- tibuscrape does not talk directly to the cloud service; rather, it relies on a local copy of the TiBU backup directory usually created by a local cloud sync agent or synced copies created by rclonesync or rclone.
- Configure the `TIBU_PATH` and `ARCHIVE_PATH` vars in the script.
- Manually create the target archive directory.
- Periodically, manually delete older archived app versions and their data files.  File creation dates are preserved in the archive, so its obvious which files are older when sorting by date.
- Consider setting up a CRON job to run tibuscrape some time after your scheduled Titanium Backup and Dropbox sync run.  Example CRON:

```
#        Minute  Hour    Day of Month   Month              Day of Week       Command    
#        (0-59)  (0-23)  (1-31)         (1-12 or Jan-Dec)  (0-6 or Sun-Sat)
           05      05      *              *                  *               (date; /<mypath to>/tibuscrape --verbose) >> /<mypath to>/runlog 2>&1
```
## CLI

(on Windows:  `> py tibuscrape -h`)

```
$ ./tibuscrape -h
usage: tibuscrape.py [-h] [-n] [-v]

Titanium Backup Scraper.

optional arguments:
  -h, --help     show this help message and exit
  -n, --dry-run  Go through the motions, but copy/delete no files.
  -v, --verbose  Print activity messages.
```

## Example run
```
Saving new  <co.happybits.marcopolo-f79465008ceaa147480058f995d5e951.apk.gz>
Saving new  <com.asynchrony.emerson.sensi-094613883e8993402bdccc3844f422bd.apk.gz>
Saving new  <dji.go.v4-a4c739c7d6565012736fed7ac7b51ed2.apk.gz>
Saving new  <com.garmin.android.apps.connectmobile-a33eb6f1344892bf850cc8d69664be75.apk.gz>
Saving new  <spinninghead.carhome-4ed856b57737c206e7483358e6e40f0f.apk.gz>
Saving new  <org.prowl.torque-abe053808d26379578c999963e24e31c.apk.gz>
Saving new  <com.eclipsim.gpsstatus2-515c7e3854d9d50fdc85b2376b12fe06.apk.gz>
Removing not latest .properties and .tar.gz files for </<mypath to>/TiBuScrapeArchive/org.openintents.notepad-20181231-091734>
Saving latest       .properties and .tar.gz files for </<mypath to>/TiBU/org.openintents.notepad-20190101-091923>
Removing not latest .properties and .tar.gz files for </<mypath to>/TiBuScrapeArchive/at.increase.wakeonlan-20181231-091740>
Saving latest       .properties and .tar.gz files for </<mypath to>/TiBU/at.increase.wakeonlan-20190101-091935>
...
Saving latest       .properties and .tar.gz files for </<mypath to>/TiBU/com.asynchrony.emerson.sensi-20190101-091927>
Saving latest       .properties and .tar.gz files for </<mypath to>/TiBU/dji.go.v4-20190101-090731>
Removing not latest .properties and .tar.gz files for </<mypath to>/TiBuScrapeArchive/com.coinbase.android-20181231-090529>
Saving latest       .properties and .tar.gz files for </<mypath to>/TiBU/com.coinbase.android-20190101-090517>
Final talley:
     8 new .apks saved 
    16 deleted .properties / .tar.gz pairs 
    24   saved .properties / .tar.gz pairs

```
## Known issues
- No error is produced if the TIBU_PATH directory is not found.  No TiBU files will be found so the script simply exits.
- If the ARCHIVE_PATH directory does not exist or temporarily cannot be reached the script will likely crash ungracefully.
- Is there value in allowing TIBU_PATH and ARCHIVE_PATH to be specified on the command line?

## Revision history
- 190101 New

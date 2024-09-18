---
title: "MS SQL Server Backups to S3 - On Linux!"
date: "2017-12-06"
categories: 
  - "dev-ops"
  - "system-administration"
---

Today I'm going to go over what is necessary in order to do full and transaction log backups for SQL Server Express on Linux. One of the big limitations of SQL Express is that it doesn't include the SQL Agent, so most of the maintenance tasks that can normally be designed and implemented within SSMS need to be rethought. Thankfully Microsoft released `sqlcmd` for Linux, which makes it pretty easy to go ahead and do the backups as simple bash scripts scheduled through cron.

## Prerequisites

This post isn't going to go through all of the steps to install SQL Server and the associated tools, but Microsoft has done a great job of documenting that on their docs site. In order to push the backups to S3 we will need the `s3cmd` tool:

```
apt install s3cmd
s3cmd --configure
```

You'll need to have an IAM identity with at least enough permissions to write to the S3 bucket you designate in the script. In the configure prompts include the keys and specify what region you want to default to.

## The Scripts

In order to do the backups, two scripts are necessary: one for the full backups and one for the transaction log backups. I've opted for a very simple structure since I only care about one database, it shouldn't be very hard to modify the script to generate backups for each database, but I'll leave that as an exercise for the reader :).

### Full Database Backups (fullBackup.sh)

```
TIMESTAMP=$(date +"%F")
BACKUP_DIR="/var/opt/mssql/backup/$TIMESTAMP"
SA_USER="SA"
SA_PASS="<Your_SA_User_Password>"

mkdir -p "$BACKUP_DIR"

chown -R mssql:mssql $BACKUP_DIR

sqlcmd -S localhost -Q "BACKUP DATABASE [<DBNAME>] TO DISK = N'$BACKUP_DIR/<DBNAME>.bak' WITH NOFORMAT, NOINIT, SKIP, NOREWIND, STATS=10" -U $SA_USER -P $SA_PASS

s3cmd put "$BACKUP_DIR/<DBNAME>.bak" "s3://<BUCKET_NAME>/$TIMESTAMP/<DBNAME>.bak"
rm -f "$BACKUP_DIR/<DBNAME>.bak"
```

### Transaction Log Backups (logBackup.sh)

```
DATESTAMP=$(date +"%F")
TIMESTAMP=$(date +"%H%M%S")
BACKUP_DIR="/var/opt/mssql/backup/$DATESTAMP/logs/$TIMESTAMP"
SA_USER="SA"
SA_PASS="<Your_SA_User_Password>"

mkdir -p "$BACKUP_DIR"

chown -R mssql:mssql $BACKUP_DIR

sqlcmd -S localhost -Q "BACKUP LOG [<DBNAME>] TO DISK = N'$BACKUP_DIR/<DBNAME>_log.bak' WITH NOFORMAT, NOINIT, SKIP, NOREWIND, NOUNLOAD, STATS=5" -U SA -P $SA_PASS

s3cmd put "$BACKUP_DIR/<DBNAME>_log.bak" "s3://<BUCKET_NAME>/$DATESTAMP/logs/$TIMESTAMP/<DBNAME>_log.bak"

rm -f "$BACKUP_DIR/<DBNAME>_log.bak"
```

Then schedule them in cron:

```
0 0 * * * /root/bin/fullBackup.sh
*/15 * * * * /root/bin/logBackup.sh
```

With the default schedule I have, full backups are taken at midnight and transaction log backups are taken every 15 minutes.

## S3 Lifecycles

While the scripts do a good job of cleaning up after themselves, S3 will (by design) never delete your data unless you specifically tell it to. S3 has a nifty feature called "Lifecycles" which allows us to specify rules for object retention (it is a powerful feature that can be used for a number of other things as well). To access it go to the AWS Console and enter into your S3 bucket. Follow these steps to setup object retention: 1. Select the Management Tab 2. Select Lifecycle 3. Click + Add lifecycle rule 4. Name the rule something descriptive ("Expire all files"). Leave the prefix blank 5. Leave Configure transition blank 6. In Expiration set the following options: ![S3 Lifecycle Creation](images/aws-lifecycle-create.png) 7. Click Save

## That's All

At this point we have full and transaction log backups configured, being pushed off site to Amazon S3. These backups are soft deleted after 7 days and fully deleted after 14 days.

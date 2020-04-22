# Migration procedure

* 1\. [Create database and user in the server MySql/MariaDB](#1-create-database-and-user-in-the-server-mysqlmariadb)
    * 1.1\. [A Single Database](#11-a-single-database)
    * 1.2\. [In Multiple DataBases or Servers MySql/MariaDB](#12-in-multiple-databases-or-servers-mysqlmariadb)
* 2\. [Download Script and install dependencies](#2-download-script-and-install-dependencies)
* 3\. [Create and prepare tables](#3-create-and-prepare-tables)
* 4\. [Data Migration](#4-data-migration)
    * 4.1\. [Data Migration (*Single Database*)](#41-data-migration-single-database)
    * 4.2\. [Data Migration (*Multiple DataBases or Servers MySql/MariaDB*)](#42-data-migration-multiple-databases-or-servers-mysqlmariadb)
* 5\. [Help](#5-help)
* 6\. [F.A.Q.](#6-faq)

> This would be the procedure to migrate the Ombi databases from SQLite to a MySQL/MariaDB server.
> 
> If there is an error you can contact Discour or you can open an incident [here](https://github.com/vsc55/ombi_sqlite_mysql/issues).


## 1. Create database and user in the server MySql/MariaDB

On the MySQL/MariaDB server we will create the database and the user that we will use later.

### 1.1. A Single Database
```mysql
CREATE DATABASE IF NOT EXISTS `Ombi` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'ombi'@'%' IDENTIFIED BY 'ombi';
GRANT ALL PRIVILEGES ON `Ombi`.* TO 'ombi'@'%' WITH GRANT OPTION;
```

### 1.2. In Multiple DataBases or Servers MySql/MariaDB
```mysql
CREATE DATABASE IF NOT EXISTS `Ombi` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE DATABASE IF NOT EXISTS `Ombi_Settings` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE DATABASE IF NOT EXISTS `Ombi_External` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'ombi'@'%' IDENTIFIED BY 'ombi';
GRANT ALL PRIVILEGES ON `Ombi`.* TO 'ombi'@'%' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON `Ombi_Settings`.* TO 'ombi'@'%' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON `Ombi_External`.* TO 'ombi'@'%' WITH GRANT OPTION;
```
> _You need to GRANT ALL PRIVILEGES for every database you create._

[Go Up](#migration-procedure)

## 2. Download Script and install dependencies
1. Download the script.
    ```bash
    $ git clone https://github.com/vsc55/ombi_sqlite_mysql.git ombi_sqlite_mysql
    $ cd ombi_sqlite_mysql
    $ chmod +x *.py
    ```
2. Install the dependencies according to the operating system we use.
    ```bash
    $ apt-get install python-mysqldb    # Debian/Ubuntu
    $ emerge -va mysqlclient            # Gentoo
    $ pip install mysqlclient           # Python Pip
    ```

[Go Up](#migration-procedure)

## 3. Create and prepare tables
1. Update to the latest version of ombi.
2. Stop ombi
3. Create or Modify **database.json** to use mysql.
    ```bash
    $ python ombi_sqlite2mysql.py -c /etc/Ombi --only_db_json --host 192.168.1.100 --db Ombi --user ombi --passwd ombi
    Migration tool from SQLite to MySql/MariaDB for ombi (3.0.2) By VSC55

    Generate file "database.json":
    - Saving in (/etc/Ombi/database.json)... [✓]
    ```
4. **Only if we are going to use *Multiple DataBases* or *Multiple Servers*.**

   To be able to use multiple servers or databases we will need to manually edit **database.json**.
    ```json
    $ vi database_multi.json
    {
        "OmbiDatabase": {
            "Type": "MySQL",
            "ConnectionString": "Server=192.168.1.100;Port=3306;Database=Ombi;User=ombi;Password=ombi"
        },
        "SettingsDatabase": {
            "Type": "MySQL",
            "ConnectionString": "Server=192.168.1.100;Port=3306;Database=Ombi_Settings;User=ombi;Password=ombi"
        },
        "ExternalDatabase": {
            "Type": "MySQL",
            "ConnectionString": "Server=192.168.1.200;Port=3306;Database=Ombi_External;User=ombi;Password=ombi"
        }
    }
    ```
    > The example above will export the **"OmbiDatabase"** and **"SettingsDatabase"** databases to server **"192.168.0.100"** but to different databases on the same server, while the **"ExternalDatabase"** database will be sent to server **"192.168.1.200"**.

5. Start ombi and wait for it to create the tables.
6. We access the ombi website to finish generating the missing tables. ExternalDatabase tables are not created until they are first accessed. 
   > **No need to start the wizard, just access the web.**
7. Stop ombi.

[Go Up](#migration-procedure)


## 4. Data Migration

### 4.1. Data Migration (*Single Database*)
> For data migration we will need the file **"migration.json"** that contains the locations of the SQLite databases.
> 
> If this file does not exist, it will be created and will search the databases in the folder specified with the parameter **"--config"**.
>
>If we don't want to migrate all the data, we can generate the file **"migration.json"** with the parameter **"--only_manager_json"** and then edit it by deleting the databases we don't want to migrate.

> ### If we do not want to export OmbiExternal.
> ```bash
> $ python ombi_sqlite2mysql.py -c /etc/Ombi --only_manager_json
> Migration tool from SQLite to MySql/MariaDB for ombi (3.0.2) By VSC55
>
> Generate file "migration.json":
> - Saving in (/etc/Ombi/migration.json)... [✓]
>
> $ vi /etc/Ombi/migration.json
> ```
> Content "migration.json":
> ```json
> {
>    "OmbiDatabase": {
>       "Type":"sqlite",
>       "ConnectionString":"Data Source=/etc/Ombi/Ombi.db"
>    },
>    "SettingsDatabase": {
>       "Type":"sqlite",
>       "ConnectionString":"Data Source=/etc/Ombi/OmbiSettings.db"
>    }
> }
> ```

1. Start data migration.
    > _The script will **empty the tables** from the MySQL/MariaDB database and automatically migrate the data from SQLite to MySQL/MariaDB._

    ```bash
    $ python ombi_sqlite2mysql.py -c /etc/Ombi --host 192.168.1.100 --db Ombi --user ombi --passwd ombi
    Migration tool from SQLite to MySql/MariaDB for ombi (3.0.2) By VSC55

    Generate file "migration.json":
    - Saving in (/etc/Ombi/migration.json)... [✓]

    Generate file "database.json":
    - Saving in (/etc/Ombi/database.json)... [✓]

    MySQL > Connecting... [✓]
    Check migration.json:
    - SettingsDatabase [SQLite >> Migrate]
    - OmbiDatabase [SQLite >> Migrate]
    - ExternalDatabase [SQLite >> Migrate]

    Dump SQLite:
    - SettingsDatabase  [############################################################] 27/27
    - OmbiDatabase      [############################################################] 117106/117106
    - ExternalDatabase  [############################################################] 574/574

    Start clean tables:
    - [CLEAN ] -> ApplicationConfiguration -> rows: 3
    - [CLEAN ] -> AspNetRoles -> rows: 12
    - [CLEAN ] -> AspNetUserRoles -> rows: 18
    - [CLEAN ] -> AspNetUsers -> rows: 6
    - [CLEAN ] -> GlobalSettings -> rows: 12
    - [CLEAN ] -> NotificationTemplates -> rows: 90
    - [BACKUP] -> __EFMigrationsHistory
    - [SKIP  ] -> __EFMigrationsHistory -> rows: 3

    - Running   [############################################################] 8/8
    Clean tables [✓]

    Start Migration:
    - Preparing [############################################################] 117716/117716
    - Running   [############################################################] 117641/117641
    - Checking  [############################################################] 43/43

    MySQL > Disconnecting... [✓]
    ```
2. Start ombi and test if everything works fine.

[Go Up](#migration-procedure)


### 4.2. Data Migration (*Multiple DataBases or Servers MySql/MariaDB*)
> For data migration to multiple databases or servers we will need the file **"database_multi.json"** that contains the locations of the servers where we are going to export the data.

1. To create the file **"database_multi.json"** we will use the file **"database.json"** so we will only have to rename it.
    ```json
    $ vi database_multi.json
    {
        "OmbiDatabase": {
            "Type": "MySQL",
            "ConnectionString": "Server=192.168.1.100;Port=3306;Database=Ombi;User=ombi;Password=ombi"
        },
        "SettingsDatabase": {
            "Type": "MySQL",
            "ConnectionString": "Server=192.168.1.100;Port=3306;Database=Ombi_Settings;User=ombi;Password=ombi"
        },
        "ExternalDatabase": {
            "Type": "MySQL",
            "ConnectionString": "Server=192.168.1.200;Port=3306;Database=Ombi_External;User=ombi;Password=ombi"
        }
    }
    ```

    > We can omit the export of a database by removing it from **"database_multi.json"** or adding the property **"Skip"**.
    > The example next will export the databases **"OmbiDatabase"** and **"SettingsDatabase"** but omit **"ExternalDatabase"**.
    ```json
    $ vi database_multi.json
    {
        "OmbiDatabase": {
            "Type": "MySQL",
            "ConnectionString": "Server=192.168.1.100;Port=3306;Database=Ombi;User=ombi;Password=ombi"
        },
        "SettingsDatabase": {
            "Type": "MySQL",
            "ConnectionString": "Server=192.168.1.100;Port=3306;Database=Ombi_Settings;User=ombi;Password=ombi"
        },
        "ExternalDatabase": {
            "Type": "MySQL",
            "ConnectionString": "Server=192.168.1.200;Port=3306;Database=Ombi_External;User=ombi;Password=ombi",
            "Skip": "True"
        }
    }
    ```

    > We can also send the same database to different servers with the following configuration.
    > The example next sends databases "OmbiDatabase", "SettingsDatabase" and "ExternalDatabase" to servers 192.168.1.100 and 192.168.1.200.
    ```json
    $ vi database_multi.json
    {
        "OmbiDatabase": {
            "Type": "MySQL",
            "ConnectionString": "Server=192.168.1.100;Port=3306;Database=Ombi;User=ombi;Password=ombi"
        },
        "SettingsDatabase": {
            "Type": "MySQL",
            "ConnectionString": "Server=192.168.1.100;Port=3306;Database=Ombi_Settings;User=ombi;Password=ombi"
        },
        "ExternalDatabase": {
            "Type": "MySQL",
            "ConnectionString": "Server=192.168.1.100;Port=3306;Database=Ombi_External;User=ombi;Password=ombi"
        },
        "OmbiDatabase": {
            "Type": "MySQL",
            "ConnectionString": "Server=192.168.1.200;Port=3306;Database=Ombi;User=ombi;Password=ombi"
        },
        "SettingsDatabase": {
            "Type": "MySQL",
            "ConnectionString": "Server=192.168.1.200;Port=3306;Database=Ombi_Settings;User=ombi;Password=ombi"
        },
        "ExternalDatabase": {
            "Type": "MySQL",
            "ConnectionString": "Server=192.168.1.200;Port=3306;Database=Ombi_External;User=ombi;Password=ombi"
        }
    }
    ```
    > ### NOTE: If you want to export all the content to several servers we will have to repeat the point "Create and prepare tables" with the different servers so that all the tables are created. You will also have to modify the file **database.json** at the end of the export process before running ombi to leave a single server for each database.

2. Start data migration.
    > The script will empty the tables from the MySQL/MariaDB database and automatically migrate the data from SQLite to MySQL/MariaDB.
    ```bash
    $ python ombi_sqlite2mysql_multi.py -c /etc/Ombi
    Migration tool from SQLite to Multi MySql/MariaDB for ombi (1.0.0) By VSC55

    - Processing DataBase (OmbiDatabase)...
    -------------------

    - Saving in (/etc/Ombi/migration.json)... [✓]
    Generate file "database.json":
    - Saving in (/etc/Ombi/database.json)... [✓]

    MySQL > Connecting... [✓]
    - Reading   [############################################################] 1/1
    Read tables [✓]

    Check migration.json:
    - OmbiDatabase [SQLite >> Migrate]
    - SettingsDatabase [No Config >> Skip]
    - ExternalDatabase [No Config >> Skip]

    Dump SQLite:
    - OmbiDatabase      [############################################################] 4068/4068

    Start clean tables:
    - [CLEAN ] -> AspNetRoles -> rows: 12
    - [CLEAN ] -> AspNetUserRoles -> rows: 190
    - [CLEAN ] -> AspNetUsers -> rows: 41
    - [CLEAN ] -> Audit -> rows: 15
    - [CLEAN ] -> ChildRequests -> rows: 11
    - [CLEAN ] -> EpisodeRequests -> rows: 407
    - [CLEAN ] -> IssueCategory -> rows: 4
    - [CLEAN ] -> IssueComments -> rows: 1
    - [CLEAN ] -> Issues -> rows: 3
    - [CLEAN ] -> MovieRequests -> rows: 198
    - [CLEAN ] -> NotificationTemplates -> rows: 100
    - [CLEAN ] -> NotificationUserId -> rows: 2
    - [CLEAN ] -> RecentlyAddedLog -> rows: 2566
    - [CLEAN ] -> RequestLog -> rows: 247
    - [CLEAN ] -> RequestQueue -> rows: 6
    - [CLEAN ] -> RequestSubscription -> rows: 18
    - [CLEAN ] -> SeasonRequests -> rows: 28
    - [CLEAN ] -> TvRequests -> rows: 11
    - [CLEAN ] -> UserNotificationPreferences -> rows: 125
    - [CLEAN ] -> UserQualityProfiles -> rows: 20
    - [BACKUP] -> __EFMigrationsHistory
    - [SKIP  ] -> __EFMigrationsHistory -> rows: 36

    - Running   [############################################################] 22/22
    Clean tables [✓]

    Start Migration:
    - Preparing [############################################################] 4074/4074
    - Running   [############################################################] 4005/4005
    - Checking  [############################################################] 28/28
    Migration [✓]

    MySQL > Disconnecting... [✓]

    ----------------------------------------------------------------
    ----------------------------------------------------------------

    - Processing DataBase (SettingsDatabase)...
    -------------------

    - Saving in (/etc/Ombi/migration.json)... [✓]
    Generate file "database.json":
    - Saving in (/etc/Ombi/database.json)... [✓]

    MySQL > Connecting... [✓]
    - Reading   [############################################################] 1/1
    Read tables [✓]

    Check migration.json:
    - OmbiDatabase [No Config >> Skip]
    - SettingsDatabase [SQLite >> Migrate]
    - ExternalDatabase [No Config >> Skip]

    Dump SQLite:
    - SettingsDatabase  [############################################################] 25/25

    Start clean tables:
    - [CLEAN ] -> ApplicationConfiguration -> rows: 3
    - [CLEAN ] -> GlobalSettings -> rows: 13
    - [BACKUP] -> __EFMigrationsHistory
    - [SKIP  ] -> __EFMigrationsHistory -> rows: 4

    - Running   [############################################################] 4/4
    Clean tables [✓]

    Start Migration:
    - Preparing [############################################################] 31/31
    - Running   [############################################################] 16/16
    - Checking  [############################################################] 3/3
    Migration [✓]

    MySQL > Disconnecting... [✓]

    ----------------------------------------------------------------
    ----------------------------------------------------------------

    - Processing DataBase (ExternalDatabase)...
    -------------------

    - Saving in (/etc/Ombi/migration.json)... [✓]
    Generate file "database.json":
    - Saving in (/etc/Ombi/database.json)... [✓]

    MySQL > Connecting... [✓]
    - Reading   [############################################################] 1/1
    Read tables [✓]

    Check migration.json:
    - OmbiDatabase [No Config >> Skip]
    - SettingsDatabase [No Config >> Skip]
    - ExternalDatabase [SQLite >> Migrate]

    Dump SQLite:
    - ExternalDatabase  [############################################################] 5201/5201

    Start clean tables:
    - [CLEAN ] -> PlexEpisode -> rows: 2174
    - [CLEAN ] -> PlexSeasonsContent -> rows: 150
    - [CLEAN ] -> PlexServerContent -> rows: 349
    - [CLEAN ] -> RadarrCache -> rows: 270
    - [CLEAN ] -> SonarrCache -> rows: 43
    - [CLEAN ] -> SonarrEpisodeCache -> rows: 2194
    - [BACKUP] -> __EFMigrationsHistory
    - [SKIP  ] -> __EFMigrationsHistory -> rows: 4

    - Running   [############################################################] 8/8
    Clean tables [✓]

    Start Migration:
    - Preparing [############################################################] 5207/5207
    - Running   [############################################################] 5180/5180
    - Checking  [############################################################] 14/14
    Migration [✓]

    MySQL > Disconnecting... [✓]

    ----------------------------------------------------------------
    ----------------------------------------------------------------

    > Updating database.json...
    - Saving in (/etc/Ombi/database.json)... [✓]
    ```

3. Start ombi and test if everything works fine.

[Go Up](#migration-procedure)


## 5. Help
```bash
$ python ombi_sqlite2mysql.py -h
Migration tool from SQLite to MySql/MariaDB for ombi (3.0.4) By VSC55

Usage: ombi_sqlite2mysql.py [options]

Options:
  -h, --help            show this help message and exit
  -c CONFIG, --config=CONFIG
                        Path folder config ombi, default /etc/Ombi.
  --host=HOST           Host server MySQL/MariaDB. If not defined, a file is
                        generated with INSERT queries.
  --port=PORT           Port server MySQL/MariaDB, default 3306.
  --db=DB               Name database, default Ombi.
  --user=USER           User with access to MySQL/MariaDB, default ombi.
  --passwd=PASSWD       User password for MySQL/MariaDB, defalt empty.
  --no_backup           Disable the backup of the "__EFMigrationsHistory"
                        table.
  --save_dump           Save all query insert in the file "data_ombi.mysql".
  --force               Force clear all tables.
  --only_db_json        Only create or modify the file "database.json" with
                        the parameters that we specify.
  --only_manager_json   Only create or modify the file "migration.json" with
                        the parameters that we specify.
```

```bash
$ python ombi_sqlite2mysql_multi.py --help
Migration tool from SQLite to Multi MySql/MariaDB for ombi (1.0.0) By VSC55

Usage: ombi_sqlite2mysql_multi.py [options]

Options:
  -h, --help            show this help message and exit
  -c CONFIG, --config=CONFIG
                        Path folder config ombi, default /etc/Ombi.
  --no_backup           Disable the backup of the "__EFMigrationsHistory"
                        table.
  --force               Force clear all tables.
  --save_dump           Save all query insert in the file.
```

[Go Up](#migration-procedure)


## 6. F.A.Q.

**P: When running `pip install mysqlclient` on Windows, I get error messages about Microsoft Visual C ++ 14.0**:

S: We need to install the Microsoft Visual Building tools to make sure we can download and install dependencies for Python. This can be downloaded [here](https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=BuildTools&rel=16).

Make sure you use the following config:
![](images/faq01_img01.png)

Once installed you probably need to reboot your system for the changes to take effect.

After you have rebooted, try the code again in command prompt:
```cmd
C:\> pip install mysqlclient
```

---

**P: Errors appear in the verification of the migrated data saying that there is more data in SQLite than in MySQL or vice versa.**
```bash
- Running   [############################################################] 9242/9242
- [!!] -> __efmigrationshistory -> [SQLite (0) / MySQL (41)] = -41
- [!!] -> applicationconfiguration -> [SQLite (0) / MySQL (3)] = -3
- [!!] -> aspnetroles -> [SQLite (0) / MySQL (12)] = -12
- [!!] -> aspnetuserroles -> [SQLite (0) / MySQL (190)] = -190
- [!!] -> aspnetusers -> [SQLite (0) / MySQL (41)] = -41
- [!!] -> audit -> [SQLite (0) / MySQL (15)] = -15
- [!!] -> childrequests -> [SQLite (0) / MySQL (11)] = -11
- [!!] -> episoderequests -> [SQLite (0) / MySQL (407)] = -407
- [!!] -> globalsettings -> [SQLite (0) / MySQL (13)] = -13
- [!!] -> issuecategory -> [SQLite (0) / MySQL (4)] = -4
- [!!] -> issuecomments -> [SQLite (0) / MySQL (1)] = -1
- [!!] -> issues -> [SQLite (0) / MySQL (3)] = -3
- [!!] -> movierequests -> [SQLite (0) / MySQL (198)] = -198
- [!!] -> notificationtemplates -> [SQLite (0) / MySQL (100)] = -100
- [!!] -> notificationuserid -> [SQLite (0) / MySQL (2)] = -2
- [!!] -> plexepisode -> [SQLite (0) / MySQL (2174)] = -2174
- [!!] -> plexseasonscontent -> [SQLite (0) / MySQL (150)] = -150
- [!!] -> plexservercontent -> [SQLite (0) / MySQL (349)] = -349
- [!!] -> radarrcache -> [SQLite (0) / MySQL (270)] = -270
- [!!] -> recentlyaddedlog -> [SQLite (0) / MySQL (2566)] = -2566
- [!!] -> requestlog -> [SQLite (0) / MySQL (247)] = -247
- [!!] -> requestqueue -> [SQLite (0) / MySQL (6)] = -6
- [!!] -> requestsubscription -> [SQLite (0) / MySQL (18)] = -18
- [!!] -> seasonrequests -> [SQLite (0) / MySQL (28)] = -28
- [!!] -> sonarrcache -> [SQLite (0) / MySQL (43)] = -43
- [!!] -> sonarrepisodecache -> [SQLite (0) / MySQL (2194)] = -2194
- [!!] -> tvrequests -> [SQLite (0) / MySQL (11)] = -11
- [!!] -> usernotificationpreferences -> [SQLite (0) / MySQL (125)] = -125
- [!!] -> userqualityprofiles -> [SQLite (0) / MySQL (20)] = -20
- Checking  [############################################################] 43/43
```
S: We will have to force the elimination of the data in all the tables with the parameter `--force` as follows.
```bash
# Single Database:
$ python ombi_sqlite2mysql.py -c /etc/Ombi --force --host 192.168.1.100 --db Ombi --user ombi --passwd ombi
```
```bash
# Multiple DataBases or Servers MySql/MariaDB:
$ python ombi_sqlite2mysql_multi.py -c /etc/Ombi --force
```

---


[Go Up](#migration-procedure)

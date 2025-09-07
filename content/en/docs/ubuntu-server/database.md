---
title: "Managing databases"
linkTitle: "xDatabases"
weight: 120
# description:
---

## MariaDB

The Linux community is largely migrating over to MariaDB:
- MySQL was purchased by Oracle and many are concerned with licensing issues
  - MariaDB was made by MySQL's original development team due to these licensing issues
- In addition to backups, always have a secondary server for redundancy
- Use LVM for the partition that houses db files so you can expand the fs as needed
- Don't install MariaDB and MySQL on the same server - there might be unforeseen issues
- Use root acct for initial setup, then create new user for admin purposes


### Installation

```bash
apt install mariadb-server              # install server package
systemctl status mariadb                # verify service started
mysql_secure_installation               # install security packages

# --- Installation Q/A --- #
root passwd
unix_socket auth                        # n - has security benefits, answer n for test env
Set root password                       # y
Remove anonymous users                  # y  
Disallow root login remotely            # y - neve allow public access to db!
Remove test db and access               # y 
Reload privileges table                 # y
```

### Connection

```bash
sudo mariadb                            # connect as root - bypass passwd requirement
mariadb -u root -p                      # connect as root - requires passwd
exit                                    # exit db shell
```

### Configuration files

Config files are in `/etc/mysql`:
- Ubuntu's configuration is modularized. For example, `mariadb.cnf` points to config files in `conf.d/` and `mariadb.conf.d`
- `mariadb.cnf` explains order that config files are read:
  - `mariadb.cnf` > `conf.d/*.cnf` > `mariadb.conf.d/*` > `my.cnf`
- Config changes that also affect MySQL: Use `.cnf` extension and place in `conf.d/`
- Config changes for mariadb only: Use `.cnf` extension and place in `mariadb.conf.d/`

```bash
conf.d/                             # 
debian.cnf                          # client settings for daemon (user, host, socket location)
debian-start*                       # startup script - sets default vals, reads debian.cnf
mariadb.cnf                         # daemon config - read on startup, sets global defaults
mariadb.conf.d/                     # 
my.cnf -> /etc/alternatives/my.cnf  # standard file for daemon
my.cnf.fallback                     # 
```

### Managing dbs

Always create a admin user so you don't have to run as `root`:
- Common to capitalize reserved words to distinguish from data
- After you add or change user perms, run `FLUSH PRIVILEGES`
- Add admin on `localhost` for security so they have to log into the server first
  - `%` is wildcard in `CREATE USER` commands
- Admin can manage the db, but not users
- Create users and assign privs as `root`
  - Best practice to restrict users to database. Create db, then create specific user for that db

#### Common commands

```bash
CREATE DATABASE mysampledb;                     # create a db
SHOW DATABASES;                                 # verify and view all dbs on server
SELECT HOST, USER, PASSWORD FROM mysql.user;    # list all users
SHOW GRANTS FOR 'appuser'@'localhost'           # view grants for specific user
DROP USER 'testuser'@'localhost';               # remove user from db
```

#### User and privileges

```bash
# --- Create admin user --- #
# This admin account can manage the database - NOT users
mariadb -u root -p                                              # 1. Log in as root
CREATE USER 'admin'@'localhost' IDENTIFIED BY '<password>';     # 2. Create admin user for localhost only
CREATE USER 'admin'@'%' IDENTIFIED BY '<password>';             #    Let admin login from any machine
CREATE USER 'admin'@'10.20.30.%' IDENTIFIED BY '<password>';    #    Let admin login from any machine on network
FLUSH PRIVILEGES;                                               # 3. Reload privileges information
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost';             # 4. Grant admin all privs on everything on localhost
FLUSH PRIVILEGES;
mariadb -u admin -p                                             # 5. Log in as admin
mariadb -u admin -p<password>                                   #    Alternate syntax - no space btwn '-p' and val

# --- Create add'l users (always as root) --- #
GRANT SELECT ON *.* TO 'readonlyuser'@'localhost' IDENTIFIED BY 'password';     # Create ro user and grants w/one command
FLUSH PRIVILEGES;

# --- Change root password --- #
ALTER USER 'root'@'localhost' IDENTIFIED BY 'newpassword';
FLUSH PRIVILEGES;

GRANT SELECT ON mysampledb.* TO 'appuser'@'localhost' IDENTIFIED BY 'password'; # ro privs on mysambledb objects
FLUSH PRIVILEGES;

GRANT ALL ON mysampledb.* TO 'appuser'@'localhost' IDENTIFIED BY 'password';    # all privs on mysambledb objects
GRANT DELETE ...    # delete rows from db tables
GRANT CREATE ...    # add tables to db
GRANT INSERT ...    # add rows to db table
GRANT SELECT ...    # read info from db
GRANT DROP   ...    # remove a db
FLUSH PRIVILEGES;
```

#### Database commands

```bash
USE dbname;                                                                 # select the db to work with
CREATE TABLE Employees (Name char(15), Age int(3), Occupation char(15));    # create db table
SHOW TABLES;                                                                # view all db tables
SHOW COLUMNS IN Employees;                                                  # view cols in db
INSERT INTO Employees VALUES ('Joe Smith', '26', 'Clown');                  # add data to db
SELECT * FROM Employees;                                                    # view all data in table
DELETE FROM Employees WHERE Name = 'Joe Smith';                             # search for Joe Smith in table and delete row
DROP TABLE Employees;                                                       # Delete a table from the db
DROP DATABASE mysampledb;                                                   # Delete the db
```

#### Backup and restore

Run from standard linux shell, not from db:
- `--databases` option includes `CREATE DATABASE` command in backup file

```bash
mysqldump -u admin -p --databases mysampledb > mysampledb.sql       # backup db
mariadb -u admin -p < mysampledb.sql                                # restore db
```

### Secondary servers

A secondary server gives you redundancy--if your first server fails, apps can still use your db:
- Requires another system with `mariadb-server` running
- Does not sync users, only syncs data
- Can create a backup, but backups become stale quickly
- Secondary server is a copy that is always up-to-date.
- Not a replacement for backups. You still need backups
- set up multiple secondary dbs with unique `server-id` in `/etc/mysql/conf.d/mysql.cnf`

Primary config:
- enable binary logging - binary logs record all changes to a db. You can transfer these to a secondary server

```bash
# 1. --- Primary server --- #
vim /etc/mysql/conf.d/mysql.cnf                 # 1. Enable binary logging
[mysql]

[mysqld]
log-bin
binlog-do-db=mysampledb
server-id=1

vim /etc/mysql/mariadb.conf.d/50-server.conf    # 2. Set bind-address to 0s to connect from other machines
...                                             #    (was '127.0.0.1')
bind-address            = 0.0.0.0
...

mariadb -u root -p                              # 3. Get MariaDB root shell
GRANT REPLICATION SLAVE ON *.* TO 'replicate'@'<slave-ip>' IDENTIFIED BY 'password';   # 4. Create user named replication that can connect from secondary server IP

systemctl restart mariadb                       # 4. Restart daemon so changes take effect
FLUSH TABLES WITH READ LOCK;                    # 5. Lock db and prevent changes/writes while config other server
mysqldump -u admin -p --databases mysampledb > mysampledeb.sql  # 6. Backup primary server
rsync -av mysampledeb.sql maria@10.20.30.41:    # 7. Transfer backup file to remote server /home dir

# 2. --- Secondary server --- #
mariadb -u root -p < mysampledb.sql             # 1. Add db file to secondary db
vim /etc/mysql/conf.d/mysql.cnf                 # 2. Add server ID to secondary db
[mysql]

[mysqld]
server-id=2

systemctl restart mariadb                       # 3. Restart service
mariadb -u root -p                              # 4. Get root db shell
CHANGE MASTER TO MASTER_HOST="192.168.56.52", MASTER_USER='replicate', MASTER_PASSWORD='password';  # 5. Associate this server with primary db

# 3. --- Primary server --- #
mariadb -u root -p                              # 1. Get root db shell
UNLOCK TABLES;                                  # 2. Unlock primary server tables

# 4. --- Secondary server --- #
SHOW SLAVE STATUS \G;                           # 3. Check slave status - must say 'Waiting for master to send event'
START SLAVE;                                    #    ONLY if #3 fails
SHOW SLAVE STATUS \G;                           #    ONLY if START SLAVE was required


# --- Verify setup --- #
# primary sever
INSERT INTO Employees VALUES ('Optimus Prime', '100', 'Transformer');   # insert new info

# secondary server
SELECT * FROM Employees;                                                # should contain new row
```

#### Troubleshooting

Try these steps if your dbs aren't synchronized:
- Make sure primary is listening for connections on the right addr:port
- For `SLAVE STATUS` errors, flush privs on primary 
- If primary and secondary won't sync, manually create the db and tables on the secondary. This should help them sync
- On secondary, manually sync with `STOP SLAVE;`, `START SLAVE;` commands

```bash
sudo ss -tulpn | grep mariadb           # verify listening for connectiosn on 0.0.0.0:3306
                                        # then, check /etc/mysql/mariadb.conf.d/50-server.cnf file
                                        # then, reboot if necessary

# SHOW SLAVE STATUS errors
FLUSH PRIVILEGES;                       # run again on primary server

# manually sync on secondary
STOP SLAVE;
START SLAVE;
```
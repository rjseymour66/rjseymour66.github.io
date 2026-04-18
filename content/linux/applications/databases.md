+++
title = 'Databases'
date = '2025-09-07T19:01:12-04:00'
weight = 10
draft = false
+++


## MariaDB

The Linux community has largely migrated to MariaDB. Oracle acquired MySQL in 2010,
and concerns about licensing led the original MySQL development team to fork the
project as MariaDB.

Follow these guidelines before you install:

- Use LVM for the partition that stores database files so you can expand the filesystem as needed.
- Do not install MariaDB and MySQL on the same server — package and port conflicts can cause unexpected behavior.
- Use the root account only for initial setup, then create a dedicated admin user.
- Beyond backups, run a secondary server for redundancy so your database remains available if the primary fails.


### Installation

Install MariaDB and secure it:

1. Install the server package.
   ```bash
   apt install mariadb-server
   ```
2. Verify the service started.
   ```bash
   systemctl status mariadb
   ```
3. Run the security hardening script.
   ```bash
   mysql_secure_installation
   ```

`mysql_secure_installation` walks you through an interactive setup. Use these recommended responses:

| Prompt | Answer | Notes |
|:---|:---|:---|
| Switch to unix_socket authentication | n | Recommended for test environments |
| Set root password | y | |
| Remove anonymous users | y | |
| Disallow root login remotely | y | Never allow public access to the database |
| Remove test database and access | y | |
| Reload privilege tables | y | |

### Connection

Use either method to connect as root. `sudo mariadb` bypasses the password prompt;
`mariadb -u root -p` requires the root password:

```bash
sudo mariadb          # connect as root without a password
mariadb -u root -p    # connect as root with a password
exit                  # exit the MariaDB shell
```

### Configuration files

Configuration files are in `/etc/mysql`. Ubuntu modularizes MariaDB configuration:
`mariadb.cnf` includes files from `conf.d/` and `mariadb.conf.d/`. Files are read in
this order: `mariadb.cnf` > `conf.d/*.cnf` > `mariadb.conf.d/*` > `my.cnf`.

Place custom configuration files based on scope:

| Scope | Directory |
|:---|:---|
| Changes that also affect MySQL | `conf.d/` |
| Changes for MariaDB only | `mariadb.conf.d/` |

Key files in `/etc/mysql`:

```bash
conf.d/                             # shared MySQL/MariaDB configuration directory
debian.cnf                          # client settings for the daemon (user, host, socket location)
debian-start*                       # startup script - sets default values, reads debian.cnf
mariadb.cnf                         # main daemon config - read on startup, sets global defaults
mariadb.conf.d/                     # MariaDB-only configuration directory
my.cnf -> /etc/alternatives/my.cnf  # standard configuration file for the daemon
my.cnf.fallback                     # fallback configuration if my.cnf is missing
```

## Managing databases

Always create an admin user so you do not have to run as `root`. Follow these conventions when managing databases:

- Capitalize SQL reserved words to distinguish them from data values.
- After you add or change user permissions, run `FLUSH PRIVILEGES`.
- Create admin users scoped to `localhost` so they must log into the server before accessing the database. Use `%` as a wildcard in `CREATE USER` commands to allow connections from any host.
- The admin account can manage database objects, but cannot create or manage other users. Use `root` for user management.
- Restrict each user to a specific database. Create the database first, then create a dedicated user for it.

### Common commands

Use these commands for common database management tasks:

```bash
CREATE DATABASE mysampledb;                     # create a database
SHOW DATABASES;                                 # list all databases on the server
SELECT HOST, USER, PASSWORD FROM mysql.user;    # list all users
SHOW GRANTS FOR 'appuser'@'localhost';          # view grants for a specific user
DROP USER 'testuser'@'localhost';               # remove a user from the server
```

### Create an admin user

The admin account can manage database objects but cannot manage other users. Run these commands as `root`:

1. Log in as root.
   ```bash
   mariadb -u root -p
   ```
2. Create the admin user scoped to the appropriate host.
   ```bash
   CREATE USER 'admin'@'localhost' IDENTIFIED BY '<password>';      # localhost only
   CREATE USER 'admin'@'%' IDENTIFIED BY '<password>';              # any host
   CREATE USER 'admin'@'10.20.30.%' IDENTIFIED BY '<password>';     # any host on a subnet
   ```
3. Grant privileges and reload the privilege table.
   ```bash
   GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost';
   FLUSH PRIVILEGES;
   ```
4. Log in as admin to verify.
   ```bash
   mariadb -u admin -p               # prompts for password
   mariadb -u admin -p<password>     # password inline, no space after -p
   ```

### Create additional users as root

You can combine user creation and privilege grants in a single `GRANT` statement:

```bash
GRANT SELECT ON *.* TO 'readonlyuser'@'localhost' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
```

Restrict users to a single database rather than granting server-wide access:

```bash
GRANT SELECT ON mysampledb.* TO 'appuser'@'localhost' IDENTIFIED BY 'password';  # read-only access
GRANT ALL ON mysampledb.* TO 'appuser'@'localhost' IDENTIFIED BY 'password';     # full access
FLUSH PRIVILEGES;
```

### Common privilege types

```bash
GRANT SELECT ...    # read rows from a table
GRANT INSERT ...    # add rows to a table
GRANT DELETE ...    # delete rows from a table
GRANT CREATE ...    # add tables to a database
GRANT DROP   ...    # remove a database
```

### Change the root password

1. Log in as root.
   ```bash
   mariadb -u root -p
   ```
2. Update the password and reload the privilege table.
   ```bash
   ALTER USER 'root'@'localhost' IDENTIFIED BY 'newpassword';
   FLUSH PRIVILEGES;
   ```

### Database commands

Use these commands to manage tables and data within a database:

```bash
USE dbname;                                                                 # switch to a database
CREATE TABLE Employees (Name char(15), Age int(3), Occupation char(15));    # create a table
SHOW TABLES;                                                                # list all tables in the database
SHOW COLUMNS IN Employees;                                                  # list columns in a table
INSERT INTO Employees VALUES ('Joe Smith', '26', 'Clown');                  # insert a row
SELECT * FROM Employees;                                                    # view all rows in a table
DELETE FROM Employees WHERE Name = 'Joe Smith';                             # delete a row by value
DROP TABLE Employees;                                                       # delete a table
DROP DATABASE mysampledb;                                                   # delete the database
```

## Backup and restore

Run these commands from the Linux shell, not from the MariaDB shell. The `--databases`
option includes a `CREATE DATABASE` statement in the backup file, which simplifies restores:

```bash
mysqldump -u admin -p --databases mysampledb > mysampledb.sql       # back up a database
mariadb -u admin -p < mysampledb.sql                                # restore a database
```

## Secondary servers

A secondary server provides redundancy. If the primary server fails, applications can
still access your database. Note that secondary servers sync data only — they do not
sync users.

A secondary server is always up-to-date, but it is not a replacement for backups. Enable
binary logging on the primary so all database changes are recorded and can be transferred
to secondary servers.

Follow these guidelines before configuring replication:

- The secondary server must have `mariadb-server` installed and running.
- Assign each secondary server a unique `server-id` in `/etc/mysql/conf.d/mysql.cnf`.

### Configure the primary server

1. Enable binary logging by adding the following to `/etc/mysql/conf.d/mysql.cnf`:
   ```ini
   [mysqld]
   log-bin
   binlog-do-db=mysampledb
   server-id=1
   ```
2. Set `bind-address` to `0.0.0.0` in `/etc/mysql/mariadb.conf.d/50-server.conf` so the server accepts connections from other machines (the default is `127.0.0.1`).

3. Create a replication user in the MariaDB shell:
   ```bash
   mariadb -u root -p
   GRANT REPLICATION SLAVE ON *.* TO 'replicate'@'<secondary-ip>' IDENTIFIED BY 'password';
   ```
4. Restart the service, lock the tables, back up the database, and transfer the backup to the secondary server:
   ```bash
   systemctl restart mariadb
   FLUSH TABLES WITH READ LOCK;
   mysqldump -u admin -p --databases mysampledb > mysampledb.sql
   rsync -av mysampledb.sql maria@10.20.30.41:
   ```

### Configure the secondary server

1. Import the backup:
   ```bash
   mariadb -u root -p < mysampledb.sql
   ```
2. Add a unique server ID to `/etc/mysql/conf.d/mysql.cnf`:
   ```ini
   [mysqld]
   server-id=2
   ```
3. Restart the service and associate the secondary with the primary:
   ```bash
   systemctl restart mariadb
   mariadb -u root -p
   CHANGE MASTER TO MASTER_HOST="192.168.56.52", MASTER_USER='replicate', MASTER_PASSWORD='password';
   ```

### Complete setup on the primary server

Unlock the tables after the secondary is configured:

```bash
mariadb -u root -p
UNLOCK TABLES;
```

### Verify replication

On the secondary server, check replication status. The output must show `Waiting for master to send event`:

```bash
SHOW SLAVE STATUS \G;
START SLAVE;          # run only if SHOW SLAVE STATUS reports an error
SHOW SLAVE STATUS \G; # run only if START SLAVE was required
```

To confirm data is syncing, insert a row on the primary and verify it appears on the secondary:

```bash
# primary server
INSERT INTO Employees VALUES ('Optimus Prime', '100', 'Transformer');

# secondary server
SELECT * FROM Employees;
```

### Troubleshooting

If your databases are not synchronized, work through these steps:

1. Verify the primary server is listening on `0.0.0.0:3306`:
   ```bash
   sudo ss -tulpn | grep mariadb
   ```
   If not, check `/etc/mysql/mariadb.conf.d/50-server.cnf` and restart the service.
2. For `SHOW SLAVE STATUS` errors, run `FLUSH PRIVILEGES` again on the primary server.
3. If the primary and secondary will not sync, manually create the database and tables on the secondary to give replication a consistent starting point.
4. To manually resync on the secondary:
   ```bash
   STOP SLAVE;
   START SLAVE;
   ```
---
title: "MySQL"
linkTitle: "MySQL"
weight: 10
description: >
  Getting up and running with MySQL.
---



### MySQL service

The MySQL service must be running to access a database. The following commands check the service, restart the service, and then configure the service to start each time the computer boots:

```shell
# check status
$ sudo systemctl status mysql

# start service
$ sudo systemctl start mysql

# stop service
$ sudo systemctl stop mysql

# restart service
$ sudo systemctl restart mysql

# start at boot
$ sudo systemctl enable mysql
```

MySQL writes errors and startup events to `/var/log/mysql/error.log`. Check this file when diagnosing connection or startup failures.

### Access a MySQL database

Access the MySQL server with the `mysql` CLI utility:

```shell
$ mysql -u <username> -p
# optional database name
$ mysql -u <username> -p <database>
```

The `-u` flag indicates the MySQL username, and the `-p` flag makes the CLI utility prompt you for your password after you enter the command. You can also provide the password directly after the `-p` flag, with no space between the flag and the value.

> Store the password in an environment variable if you pass it on the command line.

To connect to a remote MySQL server, specify `--host` and `--port`:

```shell
$ mysql -u <username> -p --host=db.example.com --port=3306
```

To enforce an encrypted connection, pass `--ssl-mode=REQUIRED`. This requires the server to have TLS (Transport Layer Security) configured:

```shell
$ mysql -u <username> -p --host=db.example.com --ssl-mode=REQUIRED
```

For details about all connection options, refer to the [documentation](https://dev.mysql.com/doc/refman/8.0/en/connecting.html).

After you connect to the MySQL server, you can view all databases:

```sql
mysql> create database golang;
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| golang             |
| grafana            |
| information_schema |
| mysql              |
| performance_schema |
| sakila             |
| sys                |
+--------------------+
7 rows in set (0.00 sec)

mysql> use golang;
Database changed
```

### User administration

To view your current database:

```mysql
mysql> SELECT DATABASE();
+------------+
| DATABASE() |
+------------+
| golang     |
+------------+
1 row in set (0.00 sec)

```

To view details about your current session, enter `status`:

```mysql
> status;
--------------
mysql  Ver 8.0.33-0ubuntu0.22.04.2 for Linux on x86_64 ((Ubuntu))

Connection id:          24
Current database:
Current user:           root@localhost
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:         8.0.33-0ubuntu0.22.04.2 (Ubuntu)
Protocol version:       10
Connection:             Localhost via UNIX socket
Server characterset:    utf8mb4
Db     characterset:    utf8mb4
Client characterset:    utf8mb4
Conn.  characterset:    utf8mb4
UNIX socket:            /var/run/mysqld/mysqld.sock
Binary data as:         Hexadecimal
Uptime:                 2 days 11 hours 33 min 56 sec

Threads: 2  Questions: 30  Slow queries: 0  Opens: 147  Flush tables: 3  Open tables: 66  Queries per second avg: 0.000
--------------

```

For comprehensive instructions on granting privileges, refer to [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql).

To create a user, enter the following:

```sql
> CREATE USER '<username>'@'localhost' IDENTIFIED BY '<password>';
```

To grant that user permissions to a specific table:

```sql
> GRANT PRIVILEGE ON database.table TO '<username>'@'host';
```

For example:

```sql
-- grant specific privileges
GRANT CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on *.* TO '<username>'@'localhost' WITH GRANT OPTION;

-- grant all privileges
GRANT ALL PRIVILEGES ON *.* TO '<username>'@'localhost' WITH GRANT OPTION;
```

As a best practice, flush the grant tables to ensure that the new privileges are put into effect:

```sql
> FLUSH PRIVILEGES;
```

To confirm that privileges were granted:

```sql
> SHOW GRANTS FOR '<username>'@'host';
```

To remove a privilege, apply `REVOKE` with the same syntax as `GRANT`:

```sql
REVOKE DELETE ON database.table FROM '<username>'@'host';
```

MySQL 8.0 introduced *roles* as a way to group privileges and assign them to multiple users. To create a role, grant it privileges, and then assign it to a user:

```sql
CREATE ROLE 'app_read';
GRANT SELECT ON mydb.* TO 'app_read';
GRANT 'app_read' TO '<username>'@'localhost';
```

### Character sets and collations

MySQL distinguishes between a *character set* (the encoding that determines which characters can be stored) and a *collation* (the rules for comparing and sorting those characters). The `utf8mb4` character set stores the full Unicode range, including emoji and supplementary characters. The older `utf8` character set in MySQL only encodes three-byte sequences and cannot store emoji.

To create a database with `utf8mb4` as the default:

```sql
CREATE DATABASE mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

To migrate an existing table to `utf8mb4`:

```sql
ALTER TABLE mytable CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

To confirm the character set of an existing table:

```sql
SHOW CREATE TABLE mytable\G
```

### Initialize the database

When you connect an application to a database, initialize the database with the required structure first. For large applications, you might need version-controlled scripts or migration files. For smaller applications, create a table initialization statement within a `const`:

```go
const (
	createTableName string = `CREATE TABLE IF NOT EXISTS "interval" (
		"id" INTEGER,
		"start_time" DATETIME NOT NULL,
		"planned_duration" INTEGER DEFAULT 0,
		"actual_duration" INTEGER DEFAULT 0,
		"category" TEXT NOT NULL,
		"state" INTEGER DEFAULT 1,
		PRIMARY KEY("id")
	);`
)
```

The `database/sql` package handles type conversion between Go and SQL types automatically.

To execute DDL (Data Definition Language) statements from a script, run the `source` command from within the MySQL CLI:

```mysql
mysql> source path/to/script.sql
```

An example `script.sql`:

```sql
-- This is a comment
CREATE TABLE IF NOT EXISTS `interval` (
		`id` INTEGER,
		`start_time` DATETIME NOT NULL,
		`planned_duration` INTEGER DEFAULT 0,
		`actual_duration` INTEGER DEFAULT 0,
		`category` TEXT NOT NULL,
		`state` INTEGER DEFAULT 1,
		PRIMARY KEY(`id`)
	);
```

Alternatively, you can model SQL statements as a `const`:

```go
const (
	createTableName string = `CREATE TABLE IF NOT EXISTS "interval" (
		"id" INTEGER,
		"start_time" DATETIME NOT NULL,
		"planned_duration" INTEGER DEFAULT 0,
		"actual_duration" INTEGER DEFAULT 0,
		"category" TEXT NOT NULL,
		"state" INTEGER DEFAULT 1,
		PRIMARY KEY("id")
	);`
)
```

MySQL generates sequential primary keys with `AUTO_INCREMENT`. This differs from PostgreSQL, which relies on `SERIAL` or `GENERATED ALWAYS AS IDENTITY`:

```sql
CREATE TABLE orders (
  id          INT AUTO_INCREMENT PRIMARY KEY,
  customer_id INT NOT NULL,
  created_at  DATETIME NOT NULL,
  total       DECIMAL(10, 2) DEFAULT 0.00
);
```

For projects with multiple developers or deployment environments, a migration tool tracks schema changes as versioned files. Common options are [golang-migrate](https://github.com/golang-migrate/migrate) and [Flyway](https://flywaydb.org/).

### Backup and restore

*mysqldump* is the standard tool for creating logical backups of MySQL databases. A logical backup stores data as SQL statements rather than raw binary files, making it portable across MySQL versions and operating systems.

To back up a single database:

```shell
$ mysqldump -u <username> -p <database> > backup.sql
```

To back up all databases:

```shell
$ mysqldump -u <username> -p --all-databases > all_databases.sql
```

To restore from a backup, pipe the SQL file into the `mysql` CLI:

```shell
$ mysql -u <username> -p <database> < backup.sql
```

For InnoDB tables, pass `--single-transaction` to capture a consistent snapshot without locking the database:

```shell
$ mysqldump -u <username> -p --single-transaction <database> > backup.sql
```

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

# restart service
$ sudo systemctl start mysql

# start at boot
$ sudo systemctl enable mysql
```

### Access a MySQL database

Access mysql with the mysql CLI utility:

```shell
$ mysql -u <username> -p
# optional database name
$ mysql -u <username> -p <database>
```

The `-u` flag indicates the MySQL username, and the `-p` flag makes the CLI utility prompt you for your password after you enter the command. You can also provide the password to the command by entering it directly after the `-p` flag, with no spaces in between. 
> Use environment variables if you are passing the password to the console.

For details about all connection options, refer to the [documentation](https://dev.mysql.com/doc/refman/8.0/en/connecting.html).

After you access the mysql server, you can view all databases:

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

### Initialize the database

When you use a database with an application, you want to initialize the db with the required structure. For large applications, you might need version controlled scripts or migration files to initialize your db. For smaller apps, create a table initialization statement within a `const`:

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
The SQLite driver automatically handles conversion between Go types and SQL types.


You can run SQL scripts to easily execute DDL statements:

```shell
$ mysql> source path/to/script.sql
```
Example `script.sql`:
```sql
-- This is a comment
CREATE TABLE IF NOT EXISTS "interval" (
		"id" INTEGER,
		"start_time" DATETIME NOT NULL,
		"planned_duration" INTEGER DEFAULT 0,
		"actual_duration" INTEGER DEFAULT 0,
		"category" TEXT NOT NULL,
		"state" INTEGER DEFAULT 1,
		PRIMARY KEY("id")
	);
```
Alternately, you can model SQL statements as a `const`:

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

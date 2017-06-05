A bash script for dumping and restoring MySQL databases with one command.

## Getting started
### Client (backup) side
__1.__ This tool should be configured on a backup server with a MySQL Client __installed and running__.

Ubuntu / Debian
```bash
$ apt-get install mysql-client
```

CentOS / RHEL
```bash
$ yum install mysql
```

__2.__ We'll create a backup user on our MySQL server during the next step. Let's
add its future details to the .my.cnf in our root dir.
```bash
$ sudo echo '[client]
> user=backup
> password=<password>' > /root/.my.cnf
```

This will prevent the MySQL Client from asking the server's 
user and password each time we connect.


### Server (host) side
__3.__ Add a backup user to your mysql server:
```sql
GRANT ALL PRIVILEGES ON *.* TO 'backup@<BACKUP_SERVER_IP>' IDENTIFIED BY '<password>';
```
Here, `<password>` is the one you defined on step 2.

__4.__ So you can access your server remotely (backup --> host), 
comment out `bind-address` on the mysql server's configuration file:
```bash
$ sed -i 's|bind-address|#bind-address|' /etc/my.cnf
```

__5.__ Reload the MySQL server:
```bash 
$ service mysql reload
```
As far as the server is concerned, you're done.

## Configuration
Create a .records file in your root directory. This is where `getdb` will find the target host's address.

Each host's entry on the .records file should have the following structure:
```
shorthand,hostname|IPADDRESS
```

e.g.
```bash 
$ echo 'server1,server1.example.com|192.168.122.3' >> /root/.records
```

You can add several aliases for your host, seperating them with a comma:
```bash
$ echo 'server1,server1.example.com,sv1|192.168.122.3' >> /root/.records
```

## Usage

### Dumping
```bash
$ bash getdb dump <host> <options>
```

Your databases will be dumped on a `/var/dbs/<server_hostname>/dump` directory.

Dumping all databases from server1, which we added earlier to our .records file:
```bash
$ bash getdb dump server1 -A
```

Dumping a specific database from server1:
```bash
$ bash getdb dump server1 wordpress
```

Dumping several databases:
```bash
$ bash getdb dump server1 wordpress db2 db3
```


### Restoring
Restoring is a work in progress. It's still buggy and needs lots of work.

```bash
$ bash getdb restore <host> <options>
```

Restoring the latest __full__ dump:
```bash
$ bash getdb restore ifm -dp latest
```

Restoring the latest dump of a __single database__:
```bash
$ bash getdb restore ifm -n db_name -p latest
```

Restoring yesterday's latest dump:
```bash
$ bash getdb restore ifm -n db_name -p yesterday
```

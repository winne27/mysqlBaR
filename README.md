#Description
This are some Perl scripts for handle mysql backup and point-in-time-recovery

#Prerequisites

The following scripts needs that the package "Percona Xtrabackup" is installed on your server.
It is freeware licensed under GPL and you can get it from http://www.percona.com/software/percona-xtrabackup/.

#Installation

Copy myBackup and myRestore to a directory which is in the path. Suggestion: /usr/local/bin
Copy my-backup.cnf and my-restore.cnf to /etc or another directory of your joice.

#Create backup user in mySql Database

Create a user (default name: backup) in the mySql database. Give the following rights to him:
GRANT RELOAD, LOCK TABLES, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'backup'@'localhost' 

#Configuration

Have a look to the config file my-backup.cnf and change the options for your needs. If point-in-time
recovery is requested check if binary logging is enabled in the mysql database. This is done by the 
following option in my.cnf:
log-bin=mysql-bin

#Backup database

The base of this system are rotating backupsets. This means that after a backupset is "full"
the next backupset is used. In the case the last backupset is reached all content from the first 
backupset is erased and then is used as actual backupset. The number of backupsets can be defined
in the config file or can be changed by callup parameter.

Each backupset contains directories named "incrementxx" where xx is between 00 and 99. The directory
increment00 contains always a full backup. In the config file the number of increments can be defined. 
If it is set to 0 only a full backup is taken. If this value is grater than 0 incremental backups are
taken until the number defined in the config file is reached. In this case the backupset is closed and 
the next backupset is selected.

For starting the backup put the following to the crontab of root:
sudo -u mysql /usr/local/bin/myBackup >/var/log/mybackup.log 2>&1

#Restore database

For restoring the database a seperate directory is used. The name of this directory is defined in 
/etc/my-backup.cnf. The default is /var/lib/mysql-restore. This could changed by a calling parameter of
myRestore. Use option --help for more information.

There are two methods for restoring the database. In the first method you have to select the backup by 
identifying the backupset and the increment that should be used for restore. This is done by call parameters
of myRestore.

The second method is a point-in-time recovery of the database. Use parameter 
--stop_datetime=YYYY-MM-DD-HH-MI-SS
to determine the time of recovery until. The procedure then detects automatically the necessary backupset and 
the increments and the binary logs needed for the recovery.

In both cases after the successful recovery an alternate database instance with this restored data is started. 
This instance uses by default the config file /etc/my-restore.cnf.  By default this database is listening 
on port 3307 and on socket /var/lib/mysql-restore/basedir/mysql.sock.

Examples for starting the restore:

myRestore --backupset 2 --increment 5
myRestore --stop_datetime 2012-01-22-18-15-00

#Stopping the recovered database service

The processid of the database service can be found in /var/lib/mysql-restore/basedir/mysql.pid. With
kill -15 pid
database could be shutdown.

# MySQL-Master-Slave-Replication-using-CentOS-7


MySQL replication is a process that allows you to automatically copy data from one database server to one or more servers.
MySQL supports a number of replication topologies with Master/Slave topology being one of the most well-known topologies in which one database server acts as the master, while one or more servers act as slaves. By default, the replication is asynchronous where the master sends events that describe database modifications to its binary log and slaves request the events when they are ready.
In this tutorial, we will explain how to set up a MySQL Master/Slave replication with one master and one slave server on CentOS 7. The same steps apply for MariaDB.
This type of replication topology is best suited for deploying of read replicas for read scaling, live databases backup for disaster recovery and for analytics jobs.
Prerequisites
In this example, we are assuming that you have two servers running CentOS 7, which can communicate with each other over a private network. If your hosting provider doesn’t provide private IP addresses, you can use the public IP addresses and configure your firewall to allow traffic on port 3306 only from trusted sources.
The servers in this example have the following IPs:
Master IP: 192.168.121.59
Slave IP:  192.168.121.14
Install MySQL
The default The CentOS 7 repositories doesn’t include MySQL packages so we will install MySQL from their official Yum Repository. To avoid any issues, we will install the same MySQL version 5.7 on both servers.
Install MySQL on both the Master and Slave servers:
sudo yum localinstall https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpmsudo yum install mysql-community-server
Once the installation is completed, start the MySQL service and enable it to automatically start on boot with:
sudo systemctl enable mysqldsudo systemctl start mysqld
When MySQL server starts for the first time, a temporary password is generated for the MySQL root user. To find the password use the following grep command :
sudo grep 'temporary password' /var/log/mysqld.log
Run the mysql_secure_installation command to set your new root password and improve the security of the MySQL instance:
mysql_secure_installation
Enter the temporary root password and answer Y (yes) to all questions.
The new password needs to be at least 8-characters long and to contain at least one uppercase letter, one lowercase letter, one number, and one special character.
Configure the Master Server
First, we will configure the master MySQL server and make the following changes:
•	Set the MySQL server to listen on the private IP .
•	Set a unique server ID.
•	Enable the binary logging.
To do so open the MySQL configuration file and add the following lines in the [mysqld] section:
sudo nano /etc/my.cnf
master:/etc/my.cnf
bind-address           = 192.168.121.59
server-id              = 1
log_bin                = mysql-bin
Once done, restart the MySQL service for changes to take effect
sudo systemctl restart mysqld
The next step is to create a new replication user. Log in to the MySQL server as the root user:
mysql -uroot -p
From inside the MySQL prompt, run the following SQL queries that will create the replica user and grant the REPLICATION SLAVE privilege to the user:
CREATE USER 'replica'@'192.168.121.14' IDENTIFIED BY 'strong_password';
GRANT REPLICATION SLAVE ON *.* TO 'replica'@'192.168.121.14';
Make sure you change the IP with your slave IP address. You can name the user as you want.
While still inside the MySQL prompt, execute the following command that will print the binary filename and position.
SHOW MASTER STATUS\G
*************************** 1. row ***************************
             File: mysql-bin.000001
         Position: 1427
     Binlog_Do_DB: 
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 
1 row in set (0.00 sec)
Take note of file name, ‘mysql-bin.000001’ and Position ‘1427’. You’ll need these values when configuring the slave server. These values will probably be different on your server.
Configure the Slave Server
Like for the master server above, we’ll make the following changes to the slave server:
•	Set the MySQL server to listen on the private IP
•	Set a unique server ID
•	Enable the binary logging
Open the MySQL configuration file and edit the following lines:
sudo nano /etc/my.cnf
slave:/etc/my.cnf
bind-address           = 192.168.121.14
server-id              = 2
log_bin                = mysql-bin
Restart the MySQL service:
sudo systemctl restart mysqld
The next step is to configure the parameters that the slave server will use to connect to the master server. Login to the MySQL shell:
mysql -uroot -p
First, stop the slave threads:
STOP SLAVE;
Run the following query that will set up the slave to replicate the master:
CHANGE MASTER TOMASTER_HOST='192.168.121.59',MASTER_USER='replica',MASTER_PASSWORD='strong_password',MASTER_LOG_FILE='mysql-bin.000001',MASTER_LOG_POS=1427;
Make sure you are using the correct IP address, user name, and password. The log file name and position must be the same as the values you obtained from the master server.
Once done, start the slave threads.
START SLAVE;
Test the Configuration
At this point, you should have a working Master/Slave replication setup.
To verify that everything works as expected, we’ll create a new database on the master server:
mysql -uroot -p
CREATE DATABASE replicatest;
Login to the slave MySQL shell:
mysql -uroot -p
Run the following command to list all databases :
SHOW DATABASES;
You will notice that the database you created on the master server is replicated on the slave:
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| replicatest        |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
Conclusion
In this tutorial, we have shown you create a MySQL Master/Slave replication on CentOS 7.
Feel free to leave a comment if you have any questions.


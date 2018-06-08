Good day,

I have completed my assignment. It is a partial implementation. I did implement all the configurations manually except the user data details used in creating the EC2 instances

Application URL: http://rotimi-lab-elb-569718014.ca-central-1.elb.amazonaws.com/

I enabled MFA on my user ID and I created the following…

VPC - Rotimi-Lab-VPC

Internet gateway - Rotimi-Lab-IGW

2 Route tables - Rotimi-Lab-RT (Connected to Internet Gateway)
			   Rotimi-Lab-1-RT (internal only — not connected to Internet gateway)

4 Subnets: Two public subnets in different availability zones and two private subnets, also in different availability zones. The public subnets are connected to the routing table which is connected to the internet gateway.

Public subnets: Rotimi-Lab-PublicSubnet2 and Rotimi-Lab-PublicSubnet
Private subnets: Rotimi-Lab-PrivateSubnet3 and Rotimi-Lab-PrivateSubnet4

Network access control lists: Rotimi-Lab-NACL
I left the Rule 30 in place because without it I could not perform a yum update on the linux servers. When i Included the rule it worked fine but will not be required for the normal operation of the application.

Security group: Rotimi-Lab-SG
Allowed port 80 and 22 for web and ssh traffic into the web servers.

Launch configuration and auto-scaling groups
Created Launch configuration: Rotimi-Lab-New-LCCopy
Created Autoscaling group: Rotimi-Lab-AS
I didn’t create scaling policies since there is no load to test, so I made “desired, Min and Max” the same size of 2

Elastic Load Balancer
I created a classic elastic load balancer: Rotimi-Lab-ELB-569718014.ca-central-1.elb.amazonaws.com

RDS database:
Created a multi-instance database called electronics with username - rotimi_daramola with password - 3M9rJ6K2VhwM. 

The RDS endpoint is electronics.chntrgvaknsf.ca-central-1.rds.amazonaws.com and RDS security group is rds-launch-wizard (sg-24bd624f)

I ensured the Linux servers could connect to the database by adding Linux server security group — sg-e6e33f8d (Rotimi-Lab-SG) to the RDS security group — rds-launch-wizard (sg-24bd624f) See proof below:

From the first server
[root@ip-10-2-1-69 repo4]# mysql -h electronics.chntrgvaknsf.ca-central-1.rds.amazonaws.com -urotimi_daramola -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 1227
Server version: 5.6.39-log MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> use electronics
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MySQL [electronics]> show tables;
+-----------------------+
| Tables_in_electronics |
+-----------------------+
| admin_users           |
| category              |
| item                  |
+-----------------------+
3 rows in set (0.00 sec)

MySQL [electronics]> select * from admin_users;
+----+--------------+-------------+
| id | user_name    | password    |
+----+--------------+-------------+
|  3 | newadminuser | newpassword |
+----+--------------+-------------+
1 row in set (0.00 sec)

MySQL [electronics]>

From the second server
[root@ip-10-2-2-221 html]# mysql -h electronics.chntrgvaknsf.ca-central-1.rds.amazonaws.com -urotimi_daramola -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 1229
Server version: 5.6.39-log MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> use electronics
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MySQL [electronics]> show tables;
+-----------------------+
| Tables_in_electronics |
+-----------------------+
| admin_users           |
| category              |
| item                  |
+-----------------------+
3 rows in set (0.00 sec)

MySQL [electronics]> select * from admin_users;
+----+--------------+-------------+
| id | user_name    | password    |
+----+--------------+-------------+
|  3 | newadminuser | newpassword |
+----+--------------+-------------+
1 row in set (0.00 sec)

MySQL [electronics]>

The code is not mine. It is owned by edlangley/inventory-webapp and I have REUSED THE CODE for this lab exercise. It can be found at https://github.com/edlangley/inventory-webapp.git

I have followed his guide lines on how the application works and how to install it. I have updated the PHP file that will make a connection to the database. As I stated earlier it is a partial implementation because it did not work as it should. Anyone should be able to log in with the user ID and password and add new item, add new category and view product list. 

I found the problem in the apache error logs. See the error below.
[Tue May 22 23:50:24.120253 2018] [:error] [pid 19643] [client 10.2.2.12:21017] PHP Fatal error:  Call to undefined function mysql_connect() in /var/www/html/admin-auth.php on line 21, referer: http://rotimi-lab-elb-569718014.ca-central-1.elb.amazonaws.com/sidebar.php

It is a PHP error coming from line 21 of the admin-auth.php file. See the section of the line of code:
21                         $conn = mysql_connect("electronics.chntrgvaknsf.ca-central-1.rds.amazonaws.com:3306", "rotimi_daramola", "3M9rJ6K2VhwM") or die(mysql_error());

I think it has to do with the connection details but its being elusive to resolve. I will need more time to figure the tables in the database (electronics)

Application URL is: http://rotimi-lab-elb-569718014.ca-central-1.elb.amazonaws.com/

Edlangley guide lines...

inventory-webapp
================

A really simple PHP+MySQL web application to inventorise items in
categories with a description.

Overview
--------
This code started life as an assignment to produce an administration
back end for a classified adverts website, during a web technologies
module I did at university. 

Several years after originally writing this code, I decided I needed
something to keep a record of what electronics components I had stored
around, so I dug out this project and re-purposed it as a general
purpose inventory. You may find it useful for a similar purpose.

Installation
------------
You will need a web server which can use PHP (Such as Apache) as well
as PHP itself and MySQL installed and configured correctly. This code
is tested with Apache 2.2.22, PHP 5.3.10, and MySQL 5.5.31.

Depending upon your system, copy the PHP files in php/ to a
location served up by your web server.

As an example, the location could be: /var/www/inventory/

Currently, the header admin-auth.php which connects to the MySQL
database requires that a MySQL user 'mysqluser' with password
'mysqlpass' exists, to add such a user:

	$ mysql -u root -p <currentpasswd>
	mysql> CREATE USER 'mysqluser'@'localhost';
	mysql> SET PASSWORD FOR 'mysqluser'@'localhost' = PASSWORD('mysqlpass');

Alternatively, edit the file admin-auth.php with an appropriate
username and password to suit your MySQL installation.

*Note*: I'm using this application on OpenShift, which is all very
cloudy and fashionable at the moment. So there is a check to see
if the `OPENSHIFT_MYSQL_DB_HOST` environment variable is set, if so
the other standard OpenShift MySQL env vars are instead used to
connect to the DB.

admin-auth.php connects to a database called "electronics". You can
change the database name, if you want to inventorise
something else. Just change the name passed to the `mysql_select_db`
call in admin-auth.php, and use that database name instead in the
following commands.

Create a MySQL database called electronics on the system:

	# mysql -u root
	mysql> CREATE DATABASE electronics;
	mysql> GRANT ALL ON electronics.* TO 'mysqluser'@'localhost';
	mysql> exit
	
Then import the MySQL dump in sql-db-sample/ to that MySQL database.

	# mysql -u root electronics < /path/to/electronics.sql

Check the database is loaded properly and accessible:

	$ mysql -u mysqluser -p
	mysql> USE electronics
	mysql> SHOW tables;
	mysql> SELECT * FROM item;

That last command will also tell you the default user name and
password to log in to the site :-)

Add or remove admin users as desired:

	mysql> INSERT INTO admin_users (user_name,password) VALUES ('newadminuser','newpassword');
	mysql> DELETE FROM admin_users WHERE user_name='bob';

These MySQL operations could be done in say, PHPMyAdmin. If so,
create the database first and make sure it is selected on the left
hand sidebar, then import the .sql file.

Lastly, the title of the site displayed on most pages can be changed
by editing site_title.php.

Usage
-----
Browse to index.php, enter a username and password from the admin_users
table. After successful login you will see a table with the items in it.
From there, use of the site is all fairly self explanatory.


This is new text

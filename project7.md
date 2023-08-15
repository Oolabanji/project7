## DevOps-Website-Solution
### Preparing NFS Server
I created an EC2 instance (Red Hat Enterprise Linux 8 on AWS) on which I would setup the NFS(Network File Storage) Server.

On this server, I attached 2 EBS volumes 10GB each as external storage to the instance and I created 3 logical volumes on it through which I would attach mounts from the external web servers.

3 logical volumes lv-opt, lv-apps and lv-logs
3 mount directory /mnt/opt, /mnt/apps and /mnt/logs
Webserver content will be stores in /apps, webserver logs in /logs and /opt will be used by Jenkins
![lsblk](https://github.com/Oolabanji/test_/assets/136812420/5b34ec4e-800f-4845-a5a8-37a7d2c09121)

I took the steps taken to create logical volumes just as in project 6.

I installed nfs-server on the nfs instance and ensured that it starts on system reboot

'sudo yum -y update'

'sudo yum install nfs-utils -y'

'sudo systemctl start nfs-server.service'

'sudo systemctl enable nfs-server.service'

'sudo systemctl status nfs-server.service'

![nfs installation status](https://github.com/Oolabanji/test_/assets/136812420/8bf8907a-975d-4f83-930a-e2f9ae825d31)


I set the mount point directory to allow read and write permissions to the webserver

![setmountpoint](https://github.com/Oolabanji/test_/assets/136812420/fc099d03-452c-4d44-9a84-fecb7d863604)

I restart NFS server 

'sudo systemctl restart nfs-server'

Since I would be creating the NFS-server, web-servers and database-server all in the same subnet

I configured NFS to interact with clients present in the same subnet as shown below.



![subnetIDlocation](https://github.com/Oolabanji/test_/assets/136812420/9603f0c7-f305-400c-abc7-0ba95b881742)

'sudo vi /etc/exports'

On the vi editor, I added the lines as seen in the image below

'sudo exportfs -arv'

![nfsaccess](https://github.com/Oolabanji/test_/assets/136812420/4d6670e1-2188-4971-b32f-4ba4a6da48ad)



This is to check what port is used by NFS so I can open it in security group


![checkingport](https://github.com/Oolabanji/test_/assets/136812420/444bdfe6-0ccb-4c9f-9881-59a0db9a6455)

The following ports were opened on the NFS server

![nfsserverports](https://github.com/Oolabanji/test_/assets/136812420/7d66010b-c467-4b95-bac6-a1c62c349abc)


### Preparing Database Server

I created an Ubuntu Server on AWS which served as the Database. And i ensured its in the same subnet as the NFS-Server.

I Installed mysql-server

'sudo apt -y update'
'sudo apt install -y mysql-server'

To enter the db environment, I run

'sudo mysql'

I created a database and name it tooling

I created a database user and name it webaccess
I granted permission to webaccess user on tooling database to do anything only from the webservers subnet cidr

![dbaseserver](https://github.com/Oolabanji/test_/assets/136812420/7645c134-193e-4474-ab2a-3207f1b2f480)

### Preparing Web Servers

I created 3 RHEL EC2 instances on AWS which serves as the web servers which are in the same subnet.

A couple of configurations was done on the web servers:

configuring NFS client

deploying tooling website application

configure servers to work with database

Installing NFS-Client

![nfsclients](https://github.com/Oolabanji/test_/assets/136812420/2dfe65f6-e0b1-48cc-9d68-2eb9d2908757)


In order to connect the /var/www directory of the webserver with the /mnt/apps on nfs server. I mounted the NFS server directory to the webserver directory

![mountnfstowebserver](https://github.com/Oolabanji/test_/assets/136812420/55a02ba4-a822-4983-a699-12eec3d99289)


In order to ensure that the mounts remain intact when the server reboots. I configured the fstab directory.

'sudo vi /etc/fstab'

added the following line 172.31.9.90:/mnt/apps /var/www nfs defaults 0 0

![fstab](https://github.com/Oolabanji/test_/assets/136812420/551062cf-9e23-4ebf-84e9-238c2630a346)


### Installing Apache and Php

'sudo yum install httpd -y'

'sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm'

'sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm'

'sudo dnf module reset php'

'sudo dnf module enable php:remi-7.4'

'sudo dnf install php php-opcache php-gd php-curl php-mysqlnd'

'sudo systemctl start php-fpm'

'sudo systemctl enable php-fpm'

'setsebool -P httpd_execmem 1'

Both /var/www and /mnt/apps contains same content. This shows that both mount points are connected via NFS.

![webserver](https://github.com/Oolabanji/test_/assets/136812420/d437a3a8-2fcf-462c-857e-b40db3a3af1a)

![nfs](https://github.com/Oolabanji/test_/assets/136812420/58252f3e-4045-4226-8950-2d50bf87aa12)

I located the log folder for Apache on the Web Server and mount it to NFS server’s export for logs. I also make sure the mount point will persist after reboot. 

![mountnfsserverlog](https://github.com/Oolabanji/test_/assets/136812420/3e775a36-ecad-4b9f-944b-062ddf94b3f2)

On the /etc/fstab persist log mount point

![fstablog](https://github.com/Oolabanji/test_/assets/136812420/318cc3b0-6426-4fca-b98b-1e14c07d224d)

On the NFS Server, I added web content into the /mnt/apps directory. This contained a html folder. The same content will be present in the /var/www directory in the web server.





![tooling](https://github.com/Oolabanji/test_/assets/136812420/1b7064a4-8b52-4fd9-9b79-a57eda707a0e)

In the /var/www/html directory , I edited the already written php script to connect to the database sudo vi /var/www/html/functions.php

![connecttodatabase](https://github.com/Oolabanji/test_/assets/136812420/fe93d602-fb98-4060-9361-6e201fad77a3)


After the modification, I connected to the database server from the web server mysql -h 172.31.0.212 -u webaccess -p Password < tooling-db.sql

![connecttodatabase2](https://github.com/Oolabanji/test_/assets/136812420/658f7fc8-3672-4f1f-b56b-b14b72db49db)

![databaseserver](https://github.com/Oolabanji/test_/assets/136812420/c4c1b749-e8ba-4cfd-b162-dcb8f184e82f)

I logged in by running the following on the browser 18.117.74.68/login.php.

![propitix tooling website](https://github.com/Oolabanji/test_/assets/136812420/6318d57b-5f57-4787-8501-42cfaf32f71e)

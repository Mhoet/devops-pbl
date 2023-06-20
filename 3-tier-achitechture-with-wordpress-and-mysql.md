
## IMPLEMENTATION OF 3 Tier ARCHITECTURE USING MYSQL AND WORDPRESS IN AWS

Based on my learning, the **Three-tier Architecture**  is a client-server software architecture pattern that comprise of 3 separate layers.
1.  **Presentation Layer**  which is the user interface such as the client server or browser on my laptop.
2.  **Business Layer**  which is the backend program that implements business logic. Application or Webserver
3.  **Data Access or Management Layer** which is the layer for data storage and data access.  [Database Server](https://www.computerhope.com/jargon/d/database-server.htm)  or File System Server such as  [FTP server](https://titanftp.com/2018/09/11/what-is-an-ftp-server/), or  [NFS Server](https://searchenterprisedesktop.techtarget.com/definition/Network-File-System)

##### For my 3-Tier Setup, I will be using:
1.  A PC to serve as my client
2.  An EC2 RedHat Linux Server as the web server (This is where I will install WordPress)
3.  An EC2 RedHat Linux server as the database my (DB) server

### STEP 1 Preparing the Web Server

1.  I launched an EC2 instance with Linux that will serve as “Web Server” and Created 3 volumes in the same Availability zone as my Web Server EC2, each of 10 GiB.
![Screenshot 2023-06-18 085336](https://github.com/Mhoet/devops-pbl/assets/81827857/771d2582-be06-4a7e-8e1a-fc9325e785cb)
![Screenshot 2023-06-18 085553](https://github.com/Mhoet/devops-pbl/assets/81827857/0b76044b-1817-4fe8-95ac-389d21518bc8)

2. I attached all three volumes one by one to my Web Server EC2 instance
![Screenshot 2023-06-18 085624](https://github.com/Mhoet/devops-pbl/assets/81827857/aad539de-5777-4527-b5b5-175e3137a70f)
   
3.  I used [`lsblk`](https://man7.org/linux/man-pages/man8/lsblk.8.html)  command to inspect what block devices are attached to the server, paying attention to names of the newly created devices attached. 
	- All devices in Linux reside in /dev/ directory so I inspected it with  `ls /dev/`  to saw all 3 newly created block devices.
    
	-  I used  [`df -h`](https://en.wikipedia.org/wiki/Df_(Unix))  command to see all mounts and free space on my server
    ![Screenshot 2023-06-18 092902](https://github.com/Mhoet/devops-pbl/assets/81827857/3146eb8c-be51-4120-ab4d-1668e170840a)

4.  I used the`gdisk`  utility to create a single partition on each of the 3 disks, one after the other.
    ![Screenshot 2023-06-18 100248](https://github.com/Mhoet/devops-pbl/assets/81827857/a5a97db3-9780-481e-a29a-71bb5738404f)
5.  I then used  `lsblk`  utility to view the newly configured partition on each of the 3 disks.
![Screenshot 2023-06-18 100610](https://github.com/Mhoet/devops-pbl/assets/81827857/b335aed0-f186-4c5e-b429-21bd8d5071ba)

6.  I installed  [`lvm2`](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux))  package using  `sudo yum install lvm2`. and ran  `sudo lvmdiskscan`  command to check for available partitions.
![Screenshot 2023-06-18 093717](https://github.com/Mhoet/devops-pbl/assets/81827857/12c501a8-b683-4193-996c-a797d96f23b3)
![Screenshot 2023-06-18 093817](https://github.com/Mhoet/devops-pbl/assets/81827857/cb81fb97-5ccc-4faf-8b96-4c78a5b76cbc)

7.  I used [`pvcreate`](https://linux.die.net/man/8/pvcreate)  utility to mark each of 3 disks as physical volumes (PVs) to be used by the LVM.
![Screenshot 2023-06-18 100304](https://github.com/Mhoet/devops-pbl/assets/81827857/97846cff-38b0-4388-86fd-e1f5db84d2f0)

8.  Verified that the Physical volume had been created successfully by running  `sudo pvs`
![Screenshot 2023-06-18 100734](https://github.com/Mhoet/devops-pbl/assets/81827857/0acfc88f-1c2d-4fcb-acbf-46d0d1cd1e71)

9.  Used  [`vgcreate`](https://linux.die.net/man/8/vgcreate)  utility to add all 3 PVs to a volume group (VG) and named the VG  **webdata-vg**
10.  Verified that your VG had been created successfully by running  `sudo vgs`
![Screenshot 2023-06-18 101010](https://github.com/Mhoet/devops-pbl/assets/81827857/b90a3638-03d9-4915-8f9a-fc8ba3142df0)

11.  Used  [`lvcreate`](https://linux.die.net/man/8/lvcreate)  utility to create 2 logical volumes.  **apps-lv**  (_**Using half of the PV size**_), and  **logs-lv**  _**Using the remaining space of the PV size**_.  
12.  Verify that your Logical Volume has been created successfully by running  `sudo lvs`
![Screenshot 2023-06-18 101205](https://github.com/Mhoet/devops-pbl/assets/81827857/7b0d4121-62f5-4aff-972b-703c8548074d)

**NOTE**: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

13.  Verified the entire setup
		- ```sudo vgdisplay -v #view complete setup - VG, PV, and LV```
![Screenshot 2023-06-18 101413](https://github.com/Mhoet/devops-pbl/assets/81827857/ac197497-fa5d-42d1-b464-26c10e685748)

		- Used ```sudo lsblk``` as well, to verify the changes.
![Screenshot 2023-06-18 101458](https://github.com/Mhoet/devops-pbl/assets/81827857/dd62852e-b01f-402f-acf0-daa50bb96e3c)

14.  Using `mkfs.ext4` i formatted the logical volumes with  [ext4](https://en.wikipedia.org/wiki/Ext4)  filesystem with :
```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
![Screenshot 2023-06-18 101940](https://github.com/Mhoet/devops-pbl/assets/81827857/1801679f-c969-4126-9c44-f018749e96e1)
![Screenshot 2023-06-18 102038](https://github.com/Mhoet/devops-pbl/assets/81827857/0cafd695-6a4c-46a8-88c5-3aecca130238)


15.  I Created  **/var/www/html**  directory to store website files and **/home/recovery/logs**  to store backup of log data using :  `sudo mkdir -p`.
    
17.  Mounted  **/var/www/html**  on  **apps-lv**  logical volume using `sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

18.  Used  [`rsync`](https://linux.die.net/man/1/rsync)  utility to backup all the files in the log directory  **/var/log**  into  **/home/recovery/logs**  using `sudo rsync -av /var/log/. /home/recovery/logs/`. (_Required before mounting the file system_) 
![Screenshot 2023-06-18 102736](https://github.com/Mhoet/devops-pbl/assets/81827857/a21bac7a-277f-409e-905e-f2356a1935d7)

19.  Mounted  **/var/log**  on  **logs-lv**  logical volume using `sudo mount /dev/webdata-vg/logs-lv /var/log`. (_Note that all the existing data on /var/log will be deleted. That is why step 15 above is very important_)
20.  Restored log files back into  **/var/log**  directory using `sudo rsync -av /home/recovery/logs/log/. /var/log`
![Screenshot 2023-06-18 103126](https://github.com/Mhoet/devops-pbl/assets/81827857/3a33b9e9-18b6-4720-988f-6b635b28bef9)

21.  Update  `/etc/fstab`  file so that the mount configuration will persist after restart of the server.
The UUID of the device will be used to update the  `/etc/fstab`  file;
		- Check the UUID using `sudo blkid`
![Screenshot 2023-06-18 103325](https://github.com/Mhoet/devops-pbl/assets/81827857/b37accb7-ad6c-4ed3-a89b-256fa3c97e88)

		- Opened sudo vi  `/etc/fstab`
		- Updated  `/etc/fstab` using my own UUID.

22.  Tested the configuration and reloaded the daemon
23.  Verified my setup by running  `df -h`, the output looked like this:
![Screenshot 2023-06-18 110219](https://github.com/Mhoet/devops-pbl/assets/81827857/250d7af8-a056-4500-9617-57f10a8adea1)

### STEP 2 Web-Server Prep. Continued
1.  I updated the repository : `sudo yum -y update`   
2.  Installed wget, Apache and it’s dependencies: `sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`
    ![Screenshot 2023-06-18 110841](https://github.com/Mhoet/devops-pbl/assets/81827857/c0f6bb42-d22e-420e-9d28-118c24476326)
![Screenshot 2023-06-18 111821](https://github.com/Mhoet/devops-pbl/assets/81827857/80c6e476-4f55-408d-824c-bc3190c8f7e4)

3.  Starte Apache: `sudo systemctl enable httpd` , ` sudo systemctl start httpd `    
4.  Installed PHP and it’s depemdencies    
    ```
    sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
    sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
    sudo yum module list php
    sudo yum module reset php
    sudo yum module enable php:remi-7.4
    sudo yum install php php-opcache php-gd php-curl php-mysqlnd
    sudo systemctl start php-fpm
    sudo systemctl enable php-fpm
    setsebool -P httpd_execmem 1   ```
    
5.  Restarted Apache: `sudo systemctl restart httpd`    
6.  Downloaded wordpress and copied wordpress to  `var/www/html`  
    ```
      mkdir wordpress
      cd   wordpress
      sudo wget http://wordpress.org/latest.tar.gz
      sudo tar xzvf latest.tar.gz
      sudo rm -rf latest.tar.gz
      cp wordpress/wp-config-sample.php wordpress/wp-config.php
      cp -R wordpress /var/www/html/    
    ```    
    ![Screenshot 2023-06-18 111846](https://github.com/Mhoet/devops-pbl/assets/81827857/e49dc2bf-b0d7-4e38-aacb-95c8028d4db4)
7.  Configure SELinux Policies    
    ```sudo chown -R apache:apache /var/www/html/wordpress
      sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
      sudo setsebool -P httpd_can_network_connect=1
    ```
![Screenshot 2023-06-18 111940](https://github.com/Mhoet/devops-pbl/assets/81827857/89a6b555-2ac1-4da4-a2e4-24658a38227c)

### STEP 3 — Preparing the Database Server
1.	Launched a second RedHat EC2 instance – ‘DB Server’  
2.	Repeated the same steps as for the Web Server, but instead of  `apps-lv`, I  created  `db-lv`  and mounted it to  `/db`  directory instead of  `/var/www/html/`.
 ![Screenshot 2023-06-18 121927](https://github.com/Mhoet/devops-pbl/assets/81827857/3f51d66d-14d6-4998-ae3b-22c8fe6d9563)
![Screenshot 2023-06-18 122035](https://github.com/Mhoet/devops-pbl/assets/81827857/12f2ef7a-3127-4e39-ae1f-a5d9bd0f1cff)
![Screenshot 2023-06-18 122111](https://github.com/Mhoet/devops-pbl/assets/81827857/bdd7691b-22b5-459a-aaa1-40bb4c7d2ffa)
![Screenshot 2023-06-18 122441](https://github.com/Mhoet/devops-pbl/assets/81827857/645738ad-880c-46f6-b0a3-56eec82bc6b4)
![Screenshot 2023-06-18 122702](https://github.com/Mhoet/devops-pbl/assets/81827857/26c15730-af63-43da-b099-e4ad190c8202)
![Screenshot 2023-06-18 122712](https://github.com/Mhoet/devops-pbl/assets/81827857/aa72d732-1130-4165-bd0c-91f09a452b93)
![Screenshot 2023-06-18 122935](https://github.com/Mhoet/devops-pbl/assets/81827857/d591170e-8cbb-4b30-99c3-ce1da8b28b78)
![Screenshot 2023-06-18 123100](https://github.com/Mhoet/devops-pbl/assets/81827857/dc8c193e-a564-4886-9628-2f54961346ce)
![Screenshot 2023-06-18 123706](https://github.com/Mhoet/devops-pbl/assets/81827857/6cbc5e47-e4bc-4737-ac45-6733eb9d0114)

### STEP 4 — Installing MySQL on the DB Server EC2
1. Installed mysql server:  `sudo yum update` and `sudo yum install mysql-server`
2.	Verified the service was up and running by using  `sudo systemctl status mysqld`. It was not running so I ran: 
```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```
![Screenshot 2023-06-18 123940](https://github.com/Mhoet/devops-pbl/assets/81827857/bc25a38d-a2e9-4011-a36d-1ab22696ea98)

### STEP 5 — Configuring DB to work with WordPress
- Created new database and user following the format below:
```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```
![Screenshot 2023-06-18 125307](https://github.com/Mhoet/devops-pbl/assets/81827857/34752a66-cdb6-4d99-913d-de4c4d3b8094)

- I realized I had created the user on my **web server's**  **public ip**, so I recreated the user on the **private ip**

![Screenshot 2023-06-18 130907](https://github.com/Mhoet/devops-pbl/assets/81827857/6e068588-b6c5-4c00-825f-a82845bdf7b5)

- I opened MySQL port 3306 on DB Server EC2. For extra security, I allowed access to the DB server  only from your **Web Server’s IP address**
![Screenshot 2023-06-18 130838](https://github.com/Mhoet/devops-pbl/assets/81827857/dbd82d39-4d1e-458f-b0f7-538eb2b48128)

- I also enabled TCP port 80 in Inbound Rules configuration for my **Web Server EC2**  so that Apache could use WordPress on it.

### STEP 6 — Configuring WordPress to connect to remote database.

1.  I installed MySQL client and tested that I could connect from my **Web Server** to my **DB server** by using  `mysql-client`

![Screenshot 2023-06-18 124323](https://github.com/Mhoet/devops-pbl/assets/81827857/df6e7e33-07e6-4b44-a11f-2de1a9141bb1)

2.  Verify if you can successfully execute  `SHOW DATABASES;`  command and see a list of existing databases.
    ![Screenshot 2023-06-18 130806](https://github.com/Mhoet/devops-pbl/assets/81827857/bf941a6f-f8a4-4c83-be90-18ec955ea3ae)
    
3.  I tried to access from my browser the link to my WordPress  `http://<Web-Server-Public-IP-Address>/wordpress/` a
    
![Screenshot 2023-06-18 143618](https://github.com/Mhoet/devops-pbl/assets/81827857/e5d82444-6225-4f68-b708-9c51536acc80)

#### I was able to also successfully log in with the credentials created on my private ip. 

### See my [IMPLEMENTATION OF CLIENT SERVER ARCHITECTURE USING MYSQL](https://github.com/Mhoet/devops-pbl/blob/main/CLIENT-SERVER-ARCHITECTURE-USING-MYSQL-AWS.md)
Credit: darey.io

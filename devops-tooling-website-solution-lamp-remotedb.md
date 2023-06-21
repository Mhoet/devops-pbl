
## IMPLEMENTATION OF  LAMP STACK WITH REMOTE DATABASE and NFS SERVER IN AWS
#### Setup and technologies used:
1.  **Infrastructure**: AWS
2.  **Webserver Linux**: Red Hat Enterprise Linux 9
3.  **Database Server**: Ubuntu 20.04 + MySQL
4.  **Storage Server**: Red Hat Enterprise Linux 8 + NFS Server
5.  **Programming Language**: PHP
6.  **Code Repository**:  [GitHub](https://github.com/darey-io/tooling.git)
7. 
### STEP 1 – Preparing The NFS Server

1.  I spun up 5 EC2 instances, 4 with **RHEL Linux 9** Operating System for the and 1 **Ubuntu 20.04**.     ![Screenshot 2023-06-13 104158](https://github.com/Mhoet/devops-pbl/assets/81827857/24d034ad-358f-4543-8426-bd93c7174993)
2.  I launched the **NFS Server** instance, created 3 volumes each of 10 GiB in the same availability zone and attached them to it.
![Screenshot 2023-06-13 105759](https://github.com/Mhoet/devops-pbl/assets/81827857/0b460436-91f3-4a1f-b338-fbd13773e61f)
- I installed **LVM2** package, then partitioned and marked as physical volumes.

![Screenshot 2023-06-13 123523](https://github.com/Mhoet/devops-pbl/assets/81827857/6e6dcab5-afc0-442e-b0ee-6f52fbd55eb1)
![Screenshot 2023-06-13 130242](https://github.com/Mhoet/devops-pbl/assets/81827857/99f8221d-1547-4d27-8867-fe8fc2e6bc69)
- I added all three PVs to a volume group **webdata-vg** and created 3 logical volumes of 9gb each for `apps`, `logs` and `opt`.

![Screenshot 2023-06-13 132039](https://github.com/Mhoet/devops-pbl/assets/81827857/06a2fc77-de47-4e96-a230-921510727664)

![Screenshot 2023-06-13 132104](https://github.com/Mhoet/devops-pbl/assets/81827857/46a2582c-d457-4fa1-802b-0bf59ffc2f5d)
-   I formatted the the logival volues as  [`xfs`](https://en.wikipedia.org/wiki/XFS)    ![Screenshot 2023-06-13 133404](https://github.com/Mhoet/devops-pbl/assets/81827857/a21d1366-f222-46f3-b61f-bf08a3af6ce3)

3. I created mount points on  `/mnt`  directory for the logical volumes:  
    Mount  `lv-apps`  on  `/mnt/apps`  – To be used by webservers  
    Mount  `lv-logs`  on  `/mnt/logs`  – To be used by webserver logs  
    Mount  `lv-opt`  on  `/mnt/opt`  – To be used by Jenkins server
    ![Screenshot 2023-06-13 135613](https://github.com/Mhoet/devops-pbl/assets/81827857/bf3af49b-087c-40b4-a08c-16c3e8e8fd2c)

4.  I installed NFS server, configured it to start on reboot and made sure it was up and running with these commands.
```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```
![Screenshot 2023-06-13 135740](https://github.com/Mhoet/devops-pbl/assets/81827857/eed9925b-b2c1-444b-b389-eae27e1d737f)

![Screenshot 2023-06-13 140100](https://github.com/Mhoet/devops-pbl/assets/81827857/3c62d9d4-b8c2-4d74-b0f2-8db8d727cc07)


5.  I made sure we set up permission that will allow the Web servers to read, write and execute files on NFS and restarted the service using:
```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service
```
![Screenshot 2023-06-13 141313](https://github.com/Mhoet/devops-pbl/assets/81827857/25ee70fc-9c08-4dc6-8596-e04c765387a7)

- Next, I configured access to NFS for clients within the same subnet CIDR by doing:
```
sudo vi /etc/exports

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

Esc + :wq!

sudo exportfs -arv
```
![Screenshot 2023-06-13 142144](https://github.com/Mhoet/devops-pbl/assets/81827857/c4c0e168-03f6-4fe4-8da7-0fe0e6cb76c1)

![Screenshot 2023-06-13 142647](https://github.com/Mhoet/devops-pbl/assets/81827857/f2a82365-55e0-4465-a143-31bce6399061)

6.  I  checked the port used by NFS using `rpcinfo -p | grep nfs` and opened it in the Security Group's inbound rule as well as ports TCP 111, UDP 111 and UDP 2049.
![Screenshot 2023-06-13 143131](https://github.com/Mhoet/devops-pbl/assets/81827857/c3f18117-a187-4e1d-895a-a3976d620c35)

![Screenshot 2023-06-13 170937](https://github.com/Mhoet/devops-pbl/assets/81827857/952744ef-b8e0-42e6-9383-3c99e0aa5af9)

### STEP 2 — Configuring The Database Server
1.  I installed MySQL server
![Screenshot 2023-06-13 171718](https://github.com/Mhoet/devops-pbl/assets/81827857/7698559f-9c43-471a-9c41-2b50cc3a5374)

2.  Created a database and named it  `tooling`
3.  Created a database user and named it  `webaccess`
![Screenshot 2023-06-13 172622](https://github.com/Mhoet/devops-pbl/assets/81827857/c8fb0b4e-3c9c-4b12-9d29-c33703f12d1d)
![Screenshot 2023-06-13 173419](https://github.com/Mhoet/devops-pbl/assets/81827857/aff3faa3-6332-46cd-b9e2-3529b50a1ed5)

4. After granting permission to  user `webaccess`  on  `tooling`  database to do anything only from the webservers  `subnet cidr`, I ran a few queries to test.

![Screenshot 2023-06-13 173546](https://github.com/Mhoet/devops-pbl/assets/81827857/ef6e4352-6e9d-4ec4-88c9-f051d34e19b8)
![Screenshot 2023-06-13 173730](https://github.com/Mhoet/devops-pbl/assets/81827857/c732999c-6421-4022-a42e-164914e7c659)

### STEP 3 — Preparing The Web Servers
#### Goals for the three web servers.
-   Configured NFS client (this step was done on all three servers)
-   Deployed a Tooling application for the Web Servers into a shared NFS folder
-   Configured the Web Servers to work with a single MySQL database

1.  Launched all three web servers instances 
2.  Installed NFS client   `sudo yum install nfs-utils nfs4-acl-tools -y` on all three.

![Screenshot 2023-06-13 180622](https://github.com/Mhoet/devops-pbl/assets/81827857/d53669f9-7ded-4006-9e05-ee30250bf1fa)

3.  Mounted  `/var/www/`  and target the NFS server’s export for apps on all three.

```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
```
![Screenshot 2023-06-13 183854](https://github.com/Mhoet/devops-pbl/assets/81827857/5b326b8d-7fcb-435e-aa85-fbf876ce40ba)

4.  Verify that NFS was mounted successfully by running  `df -h`. Make sure that the changes will persist on Web Server after reboot:
![Screenshot 2023-06-13 184009](https://github.com/Mhoet/devops-pbl/assets/81827857/35a0eaf5-7867-42c7-b259-577da448cdee)

- Then `sudo vi /etc/fstab` and added following line
```
<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0
```
![Screenshot 2023-06-13 184856](https://github.com/Mhoet/devops-pbl/assets/81827857/62286846-f6ea-43b7-a19c-a36e6babbf4c)

5.  Installed  [Remi’s repository](http://www.servermom.org/how-to-enable-remi-repo-on-centos-7-6-and-5/2790/), Apache and PHP using:

```
sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo dnf module reset php

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

setsebool -P httpd_execmem 1

```
6.  Verified that Apache files and directories were available on the Web Server in  `/var/www`  and also on the NFS server in  `/mnt/apps`. Seeing the same files confirmed NFS was mounted correctly.			
 - I created a new file  `touch index.txt`  from one server and check if the same file is accessible from other Web Servers.    ![Screenshot 2023-06-13 184921](https://github.com/Mhoet/devops-pbl/assets/81827857/d68ebca2-dc61-42bb-acd7-e8a39f2d36d6)
![Screenshot 2023-06-13 194034](https://github.com/Mhoet/devops-pbl/assets/81827857/21060de7-ce2b-4d3e-9af6-f40ffab2e04b)
![Screenshot 2023-06-13 194135](https://github.com/Mhoet/devops-pbl/assets/81827857/d01de344-4724-42c3-a17b-5174b735e394)

7.  I located the log folder for Apache on the Web Server and mounted it to NFS server’s export for logs.	
-	Repeated step №4 to make sure the mount point will persist after reboot.
   ![Screenshot 2023-06-13 200047](https://github.com/Mhoet/devops-pbl/assets/81827857/fd693fbf-9cf1-4fc7-9d19-27bc0d9003f2)
 ![Screenshot 2023-06-13 195039](https://github.com/Mhoet/devops-pbl/assets/81827857/3f3a88d3-80ad-479b-8e51-65cbc8d2286d)

8.  I forked the tooling source code from  [Darey.io's Github Account](https://github.com/darey-io/tooling.git)  to my Github account. 
![Screenshot 2023-06-13 200308](https://github.com/Mhoet/devops-pbl/assets/81827857/027b9c22-76a7-4a60-a890-4c8901a1ea7b)
-	Installed git and cloned the repo.
![Screenshot 2023-06-13 200345](https://github.com/Mhoet/devops-pbl/assets/81827857/005c7c24-0605-4538-9f74-5d961a2c6ac2)

9.  Deployed the tooling website’s code to the Webserver. Making sure that the  **html**  folder from the repository is deployed to  `/var/www/html` then opened TCP port 80 on the Web Server.
    ![Screenshot 2023-06-13 201114](https://github.com/Mhoet/devops-pbl/assets/81827857/0f926ecb-07bb-4514-a3b7-a6f5fffdaead)

10. To avoid 403 Error – checked permissions to my`/var/www/html` folder and disabled SELinux `sudo setenforce 0`. 
	- To make the change permanent, I opened the config file  `sudo vi /etc/sysconfig/selinux`  and set  `SELINUX=disabled`then restrt httpd.
![Screenshot 2023-06-13 223840](https://github.com/Mhoet/devops-pbl/assets/81827857/9e3e71d1-a790-49c5-826e-d203fa5b1bee)
![Screenshot 2023-06-13 223804](https://github.com/Mhoet/devops-pbl/assets/81827857/af008078-5bca-4876-b100-ba5bc7101c1c)

11.  Updated the website’s configuration to connect to the database (in  `/var/www/html/functions.php`  file). 	![Screenshot 2023-06-13 203030](https://github.com/Mhoet/devops-pbl/assets/81827857/cadcebbb-9133-4b4c-9901-3f40534b6e46)
![Screenshot 2023-06-13 202935](https://github.com/Mhoet/devops-pbl/assets/81827857/1e0848d7-1d24-4ccb-aa88-e009808ed4d8)
		- Installed mysql client and applied `tooling-db.sql`  script to my database using this command  `mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql`
    ![Screenshot 2023-06-13 213832](https://github.com/Mhoet/devops-pbl/assets/81827857/f681ecd6-87c6-4c93-93f2-bfc1753ce17f)
![Screenshot 2023-06-13 221749](https://github.com/Mhoet/devops-pbl/assets/81827857/352e862b-429c-454b-88ea-ee9bb996236f)
-	Set mysqld.cnf bind address to anywhere ![Screenshot 2023-06-13 221321](https://github.com/Mhoet/devops-pbl/assets/81827857/d30cc888-40b6-4512-9adb-21ae4012a5b9)
![Screenshot 2023-06-13 221334](https://github.com/Mhoet/devops-pbl/assets/81827857/c2145b38-1396-4524-97c2-087ff2d7fc1b)
12.  Created in MySQL a new admin user with username:  `admin`  and password:  `password`:    
13.  Open the website in your browser  `http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php`  and made sure I could login into the website with  `admin`  user on all 3 web servers.
![Screenshot 2023-06-13 225021](https://github.com/Mhoet/devops-pbl/assets/81827857/25a23cd8-8ea2-4bc5-86df-a4dc0f60bb54)
![Screenshot 2023-06-13 225255](https://github.com/Mhoet/devops-pbl/assets/81827857/1640798d-faab-4986-9585-1f9069101af2)

### See my [IMPLEMENTATION OF 3 Tier ARCHITECTURE USING MYSQL AND WORDPRESS IN AWS](https://github.com/Mhoet/devops-pbl/blob/main/3-tier-achitechture-with-wordpress-and-mysql.md)
Credit: darey.io

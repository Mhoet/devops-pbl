# WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS
### On this project I used the AWS virtual machine EC2 t3.micro instance which provides me just enough virtual CPU, memory and bandwidth to run Linux, Apache, MySQL and PHP.

#### Steps
- I created an EC2 t3.micro instance on aws and configured it for both ssh and http clients. ![Screenshot 2023-05-12 123933](https://github.com/Mhoet/devops-pbl/assets/81827857/de511684-48bb-4872-ada2-763fc70bb829)

- I installed windows subsystem for linux (wsl) and launched the t3.micro instance with Ubuntu Server 20.04 .![Screenshot (133)](https://github.com/Mhoet/devops-pbl/assets/81827857/f19b8118-46d7-4fde-865a-960a3189b242)

 ### I installed **Apache** and updated the firewall using 
```sudo apt update```
```sudo apt install apache2```

![Screenshot (136)](https://github.com/Mhoet/devops-pbl/assets/81827857/7ee8c00d-be52-452c-84f1-be0547de854c)
- and tested that apache is running on the O.S using the command ```sudo systemctl status apache2```
![Screenshot (138)](https://github.com/Mhoet/devops-pbl/assets/81827857/23875d3f-dff3-4d74-b215-1a1dae0e4698)
- My server was running and I could access it locally on ubuntu shell as seen in the image below ![Screenshot (139)](https://github.com/Mhoet/devops-pbl/assets/81827857/87fb6dc8-5e95-437c-8ce9-f77f00252389)
 ### Next, I installed ***MySQL*** using the commands
 ```sudo apt install mysql-server```
- I set a password for root user and and removed some default setting using ```sudo mysql```
 ```ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY <settingapassword...';``` , 
 then ```sudo mysql_secure_installation``` which I used to validate my password. 
 - I was able to login to mysql using the command ```sudo mysql -p```  ![Screenshot (141)](https://github.com/Mhoet/devops-pbl/assets/81827857/27fcf909-4042-4b03-9212-17b95a7cfdcd)
 
  ### Next, I installed ***PHP***, ***php-mysql***  and ***libapache2-mod-php*** 
  - ***PHP*** processed the code to display content to user(s)
  -  ***php-mysql***  allowed PHP to communicate with MySQL installed earlier 
  - ***libapache2-mod-php*** allowed Apache to process the PHP files.
  To install all 3, I used the command ```sudo apt install php libapache2-mod-php php-mysql``` and ```php -v``` to see the version of php installed.
  ![Screenshot (142)](https://github.com/Mhoet/devops-pbl/assets/81827857/2f000d96-1706-4d4d-8f01-77865ed3bbc9)
  
## CREATING A VIRTUAL HOST FOR MY WEBSITE USING APACHE 

To create a virtual host for my website, I setup a domain called lampstack using the codes below
- Created the directory ```sudo mkdir /var/www/lampstack ```
- assigned ownership to myself (current user) ```sudo chown -R $USER:$USER /var/www/lampstack ```
- I created a config file ```sudo vi /etc/apache2/sites-available/lampstack .conf```
- Added the the following bare-bone configuration settings as seen below
```
<VirtualHost *:80>
    ServerName lampstack 
    ServerAlias www.lampstack 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/lampstack 
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- I used the ```sudo a2ensite lampstack``` command to enable the new virtual host and used ```sudo a2dissite 000-default``` to disable the default website that came with Apache.
- I then ran a config test and reloaded Apache using
```
sudo apache2ctl configtest
sudo systemctl reload apache2
```
- Since the webroot /var/www/lampstack was still empty, I created an index.html file in the same directory and added to text to it. 
 ```
sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/lampstack/index.html
```
- Next, I created an index.php file in the same directory using ```
vim /var/www/lampstack/index.php``` and added the below php code 
```
<?php
phpinfo();
```
As part of my learning, I learned that with the default **DirectoryIndex** settings on Apache, a file named `index.html` will always take precedence over an `index.php` file and to change that behavior, I’ll need to edit the **/etc/apache2/mods-enabled/dir.conf** file to change the order in which the **index.php** file is listed within the **DirectoryIndex** directive using 
```sudo vim /etc/apache2/mods-enabled/dir.conf``` like so;
```
<IfModule mod_dir.c>
        #Change this:
        #DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
        #To this:
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```
#### After saving and closing the file, I reloaded Apache so the changes take effect:
```sudo systemctl reload apache2```

By the time I reloaded my ip-address on my web server, I had this 
![Screenshot (140)](https://github.com/Mhoet/devops-pbl/assets/81827857/c1ae1d3c-e7c1-437d-ad3f-526195f27ec8)

Credit: darey.io

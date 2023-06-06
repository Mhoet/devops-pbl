# WEB STACK IMPLEMENTATION LEMP STACK (Linux, **NGINX**, MySQL and PHP.) IN AWS
### Using AWS virtual machine EC2 t3.micro instance (same as in my previous [LAMP STACK IMPLEMENTATION](https://github.com/Mhoet/devops-pbl/blob/main/lamp-stack-implementation.md))
## STEPS RECREATED
### I installed **NGINX** and updated the firewall using 
```sudo apt install nginx```
![Screenshot (145)](https://github.com/Mhoet/devops-pbl/assets/81827857/bc8932e1-4839-41aa-ac4c-d21f0b2b6939)
- and tested that apache is running on the O.S using the command ```sudo systemctl status nginx```
![Screenshot (149)](https://github.com/Mhoet/devops-pbl/assets/81827857/d5a59b7e-f400-491d-b095-b71c2d920dd6)
- My server was running and I could access it locally on ubuntu shell and on the ip address as seen in the image below !
![Screenshot (150)](https://github.com/Mhoet/devops-pbl/assets/81827857/0c4bdfe7-f0b4-4d85-a788-e00e11d4a4e1)
![Screenshot (151)](https://github.com/Mhoet/devops-pbl/assets/81827857/e17e6fa4-4030-4876-bb49-584c89b36ca0)
 ### Next, I installed ***MySQL*** using the commands
 ```sudo apt install mysql-server```
- I set a password for root user and removed some default setting using ```sudo mysql```
 ```ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY <settingapassword...';``` , 
 then ```sudo mysql_secure_installation``` which I used to validate my password. 
 - I was able to login to mysql using the command ```sudo mysql -p```  
![Screenshot (152)](https://github.com/Mhoet/devops-pbl/assets/81827857/7d2123ac-c41c-431f-a398-55202d19212d)![Screenshot (156)](https://github.com/Mhoet/devops-pbl/assets/81827857/d08e5bb4-eba2-4309-a03b-28da8a597e7a)

  ### Next, I installed ***php-mysql***  and ***php-fpm***
  - ***php-fpm*** - “PHP fastCGI process manager”, which tells Nginx what requests to pass PHP for processing
  -  ***php-mysql***  allowed PHP to communicate with MySQL installed earlier 
  To install all 2, I used the command ```sudo apt install php-fpm php-mysql```.
![Screenshot (157)](https://github.com/Mhoet/devops-pbl/assets/81827857/a0f9b367-6b6d-478c-8d3d-92d8df2d039a)
## CONFIGURING NGINX TO USE PHP PROCESSOR
I learnt during my learning that 'Nginx has one server block enabled by default and is configured to serve documents out of a directory at /var/www/html. While this works well for a single site, it can become difficult to manage if you are hosting multiple sites. Instead of modifying /var/www/html, it said to create a directory structure within /var/www for the your_domain website, leaving /var/www/html in place as the default directory to be served if a client request does not match any other sites.''
- I Created the directory ```sudo mkdir /var/www/projectLEMP ```
- Assigned ownership to myself (current user) ```sudo chown -R $USER:$USER /var/www/projectLEMP ```
- I created a config file ```sudo nano /etc/nginx/sites-available/projectLEMP```
- Added the the following bare-bone configuration settings as seen below
```
#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```
- To activate the configuration, the config file is linked to from Nginx’s sites-enabled directory using 
```sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/```
- This will tell Nginx to use the configuration next time it is reloaded. 
- To test the configuration for syntax errors I used ```sudo nginx -t``` and got a success message!
- Then to disable default Nginx host that was configured to listen on port 80, I ran ```sudo unlink /etc/nginx/sites-enabled/default```
![Screenshot (162)](https://github.com/Mhoet/devops-pbl/assets/81827857/6d0561fe-13e3-456f-9a63-6e46dee61910)
I then reloaded Nginx to apply the change using ```sudo systemctl reload nginx```
- Since the webroot /var/www/projectLEMP was still empty, I created an index.html file in the same directory and added the below text to it. 
 ```
sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```
- I opened the url using my ip-address and got this
![Screenshot (163)](https://github.com/Mhoet/devops-pbl/assets/81827857/08d2ca7d-54ff-4225-999f-d827d077d0e3)
### Testing PHP with NGINX
- Next, I created an index.php file in the same directory using ```
vim /var/www/projectLEMP/info.php``` and added the below php code 
```
<?php
phpinfo();
```
#### After saving and closing the file, I reloaded NGINX so the changes take effect using ```sudo systemctl reload nginx```
I was able to access this page in your web browser by visiting the public IP address I had set up in my Nginx configuration file, followed by /info.php:
By the time I reloaded my ip-address on my web server, I had this 
![Screenshot (164)](https://github.com/Mhoet/devops-pbl/assets/81827857/ce2db20e-94f2-444b-a743-757c7b22c7d5)
![Screenshot (165)](https://github.com/Mhoet/devops-pbl/assets/81827857/527d688d-42aa-4d3a-a1c8-f389cbc605fe)
![Screenshot (166)](https://github.com/Mhoet/devops-pbl/assets/81827857/5110578d-3610-41df-85ad-000eea05331b)
## RETRIEVING DATA FROM MYSQL DATABASE WITH PHP
- First, I connected to the MySQL console on root account using ```sudo mysql```
- To create a new database, I ran the following command from my MySQL console ```mysql> CREATE DATABASE `example_database`;```
- I created a new user and granted full privileges on the database I had created.
```mysql>  CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY '********';```
```mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';```
```sql> exit```
This gave the **example_user**  full privileges over the **example_database**, while preventing the user from creating or modifying other databases on my server.
- I logged onto mysql, viewed the database and got the below result;
![Screenshot (169)](https://github.com/Mhoet/devops-pbl/assets/81827857/5bcc942f-feff-44dd-97d7-5082bf4fb204)

- Next, I created a test table named *todo_list.* From the MySQL console and inserted a value, with the following statement:
```CREATE TABLE example_database.todo_list (item_id INT AUTO_INCREMENT,content VARCHAR(255),PRIMARY KEY(item_id));```
```mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");```
To confirm that the data was successfully saved to your table, I ran:
```mysql>  SELECT * FROM example_database.todo_list;```
And got the following output:
![Screenshot (174)](https://github.com/Mhoet/devops-pbl/assets/81827857/33480b88-b5b9-4c04-bec9-fa61e6977b70)
After confirming that I had valid data in your test table, I exited the MySQL console:
```mysql> exit```
### Creating a PHP script that connects to MySQL and query for content. 
- I created a new PHP file in my custom web root directory: ```nano /var/www/projectLEMP/todo_list.php```
I added the below content into my todo_list.php script, which connects to the MySQL database and queries for the content of the todo_list table and displays the results in a list.
```
<?php
$user = "example_user";
$password = "*******";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
} 
```
I was able to access the data I inserted into the test table on ```http://<my-ip-address>/todo_list.php```
 ![Screenshot (175)](https://github.com/Mhoet/devops-pbl/assets/81827857/a3a6dff5-e81e-4d81-ab39-12b7f977d82b)
### My PHP environment is ready to connect and interact with my MySQL server.

Credit: darey.io

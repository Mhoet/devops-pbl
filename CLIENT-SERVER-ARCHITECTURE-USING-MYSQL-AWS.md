
## IMPLEMENTATION OF CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS) IN AWS


## STEPS RECREATED (BACKEND)

1.  I created and configure two Linux-based virtual servers (EC2 instances in AWS).

```
Server A name - `mysql server`
Server B name - `mysql client`
```
![Screenshot 2023-06-18 071322](https://github.com/Mhoet/devops-pbl/assets/81827857/4bdbff38-8d44-41ba-a21b-f22ee3aa8a91)

2.  On  `mysql server`  Linux Server, I installed MySQL  **Server**  software.

![Screenshot 2023-06-18 071548](https://github.com/Mhoet/devops-pbl/assets/81827857/76f8cb4d-d2e6-40e2-a1d8-0d1222c01eff)

3.  On  `mysql client`  Linux Server, I installed MySQL  **Client**  software.   ![Screenshot 2023-06-18 071849](https://github.com/Mhoet/devops-pbl/assets/81827857/59c4104e-82c7-4e3f-a302-23a8630dc9ad)

4.  I used  `mysql server's`  local IP address to connect from  `mysql client`.  Since MySQL server uses TCP port 3306 by default, I opened it by creating a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups and allowed access only to the specific IP address of my `mysql`client’.   ![Screenshot 2023-06-18 073047](https://github.com/Mhoet/devops-pbl/assets/81827857/d64b24e7-1031-474f-9816-0235e0b4650f)
5.  I configured MySQL server to allow connections from remote hosts by running: ```
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf``` and replaced ‘127.0.0.1’ to ‘0.0.0.0’ like this:
![Screenshot 2023-06-18 073343](https://github.com/Mhoet/devops-pbl/assets/81827857/bd783542-2602-40ab-981c-d59c1d468411)

6.  From  `mysql client`  Linux Server, I connected remotely to  `mysql server`  Database Engine. I used the  `mysql`  utility to perform this action by:
	- creating a database on `mysql server` and a user at the `mysql client's`  ip  and granted the required permission.
    ![Screenshot 2023-06-18 074601](https://github.com/Mhoet/devops-pbl/assets/81827857/4d3c8d59-bffc-43d7-8cf1-d27c96886a9e)
	- I updated the firewall rules to allow connections from the `mysql client's` public ip on port 3306 and restarted `mysql`
	- I successfully connected remotely to  `mysql server` on `mysql client`
![Screenshot 2023-06-18 081741](https://github.com/Mhoet/devops-pbl/assets/81827857/e5456b7e-a45f-4cdb-801a-b2c45353162a)

7.  I also confirmed by successfully running queries on MySQL server by running  ```
Show databases;```

![Screenshot 2023-06-18 090257](https://github.com/Mhoet/devops-pbl/assets/81827857/fcd2ea35-2b8e-458b-bd2a-8c5d4493e5e0)

### See my [MEAN STACK IMPLEMENTATION](https://github.com/Mhoet/devops-pbl/blob/main/mean-stack-implementation.md)
Credit: darey.io

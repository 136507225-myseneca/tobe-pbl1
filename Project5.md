# IMPLEMENT A CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS).
## TASK – Implement a Client Server Architecture using MySQL Database Management System (DBMS).
To demonstrate a basic client-server using MySQL Relational Database Management System (RDBMS), follow the below instructions

## Create and configure two Linux-based virtual servers (EC2 instances in AWS).
Server A name - `mysql server`
Server B name - `mysql client`
On mysql server Linux Server install MySQL Server software.
Interesting fact: MySQL is an open-source relational database management system. Its name is a combination of "My", the name of co-founder Michael Widenius’s daughter, and "SQL", the abbreviation for Structured Query Language.

On mysql client Linux Server install MySQL Client software.

By default, both of your EC2 virtual servers are located in the same local virtual network, so they can communicate to each other using local IP addresses. Use mysql server's local IP address to connect from mysql client. MySQL server uses TCP port 3306 by default, so you will have to open it by creating a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups. For extra security, do not allow all IP addresses to reach your ‘mysql server’ – allow access only to the specific local IP address of your ‘mysql client’.

You might need to configure MySQL server to allow connections from remote hosts.

```javascript
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```
Replace ‘127.0.0.1’ to ‘0.0.0.0’ like this:
 

## Run this script on the Database Server
```javascript
sudo mysql_secure_installation 
sudo mysql
```
## Allow client private ip address in the inbound rules while selecting mysql


## Run this script on DB Server

```javascript
CREATE USER 'remote_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
CREATE DATABASE test_db;
GRANT ALL ON test_db.* TO 'remote_user'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```


From mysql client Linux Server connect remotely to mysql server Database Engine without using SSH. You must use the mysql utility to perform this action.

## Check that you have successfully connected to a remote MySQL server and can perform SQL queries:

```javascript
Show databases;
```
If you see an output similar to the below image, then you have successfully completed this project – you have deloyed a fully functional MySQL Client-Server set up.
Well Done! You are getting there gradually. You can further play around with this set up and practice in creating/dropping databases & tables and inserting/selecting records to and from them.

Congratulations!


## From mysql client Linux Server connect remotely to mysql server Database Engine without using SSH

```javascript
sudo mysql -u remote_user -h 172.31.47.133 (ip address of database server) -p
```

# Screenshots 

<img width="1800" alt="Screenshot 2022-04-22 at 18 12 32" src="https://user-images.githubusercontent.com/33035619/164762605-880a402f-1b6a-4254-a09f-b3c64ee4a963.png">
<img width="1800" alt="Screenshot 2022-04-22 at 18 00 35" src="https://user-images.githubusercontent.com/33035619/164762657-b010c6f0-d4d8-480e-b8cf-b4efe113593c.png">
<img width="1800" alt="Screenshot 2022-04-22 at 17 56 52" src="https://user-images.githubusercontent.com/33035619/164762671-1a27c02c-1292-442a-a8ca-4e0b2c3bf41a.png">
<img width="1800" alt="Screenshot 2022-04-22 at 17 44 25" src="https://user-images.githubusercontent.com/33035619/164762682-bca33e71-2ddd-4f48-9987-3f7a7e8dac61.png">


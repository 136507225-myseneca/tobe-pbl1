# CONFIGURE APACHE AS A LOAD BALANCER

## Configure Apache As A Load Balancer
- Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb, so your EC2 list will look like this:

Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group.
- Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:

## Install apache2

```javascript
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev
```

## Enable following modules:

```javascript
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic
```

## Restart apache2 service

```javascript
sudo systemctl restart apache2
```
Make sure apache2 is up and running

```javascript
sudo systemctl status apache2
```

Configure load balancing

```javascript
sudo vi /etc/apache2/sites-available/000-default.conf
```

## Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

```javascript
<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/
```

## Restart apache server

```javascript
sudo systemctl restart apache2
```

## ScreenShots

<img width="893" alt="Screenshot 2022-04-29 at 13 42 35" src="https://user-images.githubusercontent.com/33035619/165947779-44e13296-d7be-4c8e-9899-b818391b02f1.png">
<img width="889" alt="Screenshot 2022-04-29 at 13 47 12" src="https://user-images.githubusercontent.com/33035619/165947808-4611f43b-26d0-4590-b5ef-9980231b8c66.png">



bytraffic balancing method will distribute incoming load between your Web Servers according to current traffic load. We can control in which proportion the traffic must be distributed by loadfactor parameter.

You can also study and try other methods, like: bybusyness, byrequests, heartbeat

Verify that our configuration works – try to access your LB’s public IP address or Public DNS name from your browser:
http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php
Note: If in the Project-7 you mounted /var/log/httpd/ from your Web Servers to the NFS server – unmount them and make sure that each Web Server has its own log directory.

Open two ssh/Putty consoles for both Web Servers and run following command:

```javascript
sudo tail -f /var/log/httpd/access_log
```
  
Try to refresh your browser page http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php several times and make sure that both servers receive HTTP GET requests from your LB – new records must appear in each server’s log file. The number of requests to each server will be approximately the same since we set loadfactor to the same value for both servers – it means that traffic will be disctributed evenly between them.

If you have configured everything correctly – your users will not even notice that their requests are served by more than one server.

Side Self Study:
Read more about different configuration aspects of Apache mod_proxy_balancer module. Understand what sticky session means and when it is used.

Optional Step – Configure Local DNS Names Resolution
Sometimes it is tedious to remember and switch between IP addresses, especially if you have a lot of servers under your management.
What we can do, is to configure local domain name resolution. The easiest way is to use /etc/hosts file, although this approach is not very scalable, but it is very easy to configure and shows the concept well. So let us configure IP address to domain name mapping for our LB.

## Open this file on your LB server

```javascript
sudo vi /etc/hosts
 ```

#Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

  ```javascript
<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2
  ```
  
Now you can update your LB config file with those names instead of IP addresses.

BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1
You can try to curl your Web Servers from LB locally curl http://Web1 or curl http://Web2 – it shall work.

Remember, this is only internal configuration and it is also local to your LB server, these names will neither be ‘resolvable’ from other servers internally nor from the Internet.

Targed Architecture
Now your set up looks like this:

# ScreenShots
<img width="1800" alt="Screenshot 2022-04-27 at 23 29 13" src="https://user-images.githubusercontent.com/33035619/165949714-34a74e65-49c3-497c-8130-96faa3528fd6.png">
  

Congratulations!
You have just implemented a Load balancing Web Solution for your DevOps team.

  
  

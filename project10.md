# LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

### Task
This project consists of two parts:

- Configure Nginx as a Load Balancer
- Register a new domain name and configure secured connection using SSL/TLS certificates
  Your target architecture will look like this:
  
# CONFIGURE NGINX AS A LOAD BALANCER
You can either uninstall Apache from the existing Load Balancer server, or create a fresh installation of Linux for Nginx.

Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)
Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses
Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers
Update the instance and Install Nginx

- Update the instance and Install Nginx

```javascript
sudo apt update
sudo apt install nginx
Configure Nginx LB using Web Servers’ names defined in /etc/hosts
```

- Configure Nginx LB using Web Servers’ names defined in /etc/hosts

Hint: Read this blog to read about /etc/host

Open the default nginx configuration file

sudo vi /etc/nginx/nginx.conf

#insert following configuration into http section

```javascript
 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;
```
- Restart Nginx and make sure the service is up and running

```javascript
sudo systemctl restart nginx
sudo systemctl status nginx
```

## Screenshot
<img width="1038" alt="Screenshot 2022-05-02 at 17 14 11" src="https://user-images.githubusercontent.com/33035619/166268643-fcbe09ae-e1f3-4bdc-9a26-6d19b4317b8d.png">

<img width="1035" alt="Screenshot 2022-05-02 at 17 16 43" src="https://user-images.githubusercontent.com/33035619/166269030-a3a6cda0-683f-40b1-93af-a10aa2df07ca.png">



## REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES

Let us make necessary configurations to make connections to our Tooling Web Solution secured!

In order to get a valid SSL certificate – you need to register a new domain name, you can do it using any Domain name registrar – a company that manages reservation of domain names. The most popular ones are: Godaddy.com, Domain.com, Bluehost.com.

Register a new domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)

Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP

You might have noticed, that every time you restart or stop/start your EC2 instance – you get a new public IP address. When you want to associate your domain name – it is better to have a static IP address that does not change after reboot. Elastic IP is the solution for this problem, learn how to allocate an Elastic IP and associate it with an EC2 server on this page.

- Update A record in your registrar to point to Nginx LB using Elastic IP address

- Check that your Web Servers can be reached from your browser using new domain name using HTTP protocol – http://<your-domain-name.com>

- Configure Nginx to recognize your new domain name
Update your nginx.conf with server_name www.<your-domain-name.com> instead of server_name www.domain.com

- Install certbot and request for an SSL/TLS certificate
Make sure snapd service is active and running

```javascript
sudo systemctl status snapd
```
- Install certbot

```javascript
sudo snap install --classic certbot
```

Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file so make sure you have updated it on step 4).

```javascript
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```

Test secured access to your Web Solution by trying to reach https://<your-domain-name.com>

You shall be able to access your website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in your browser’s search string.
Click on the padlock icon and you can see the details of the certificate issued for your website.



- Set up periodical renewal of your SSL/TLS certificate
By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.

You can test renewal command in dry-run mode

```javascript
- sudo certbot renew --dry-run
```
Best pracice is to have a scheduled job that to run renew command periodically. Let us configure a cronjob to run the command twice a day.

To do so, lets edit the crontab file with the following command:

crontab -e
Add following line:

```javascript
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```

You can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.

Side Self Study: Refresh your cron configuration knowledge by watching this video.

You can also use this handy online cron expression editor.


# SceenShots
<img width="1101" alt="Screenshot 2022-05-02 at 19 06 12" src="https://user-images.githubusercontent.com/33035619/166301063-9b4de54d-8037-4946-b38a-02acec77efb6.png">

<img width="1020" alt="Screenshot 2022-05-02 at 19 47 25" src="https://user-images.githubusercontent.com/33035619/166306814-b4fc33b7-0ef7-4206-b1f7-76c02bb8fd4c.png">

<img width="1007" alt="Screenshot 2022-05-02 at 19 59 23" src="https://user-images.githubusercontent.com/33035619/166308387-7fabdbee-ef75-4e88-8278-3b5c805d62c4.png">
<img width="1004" alt="Screenshot 2022-05-02 at 20 01 51" src="https://user-images.githubusercontent.com/33035619/166308770-869cd998-32e4-483a-9af6-be868afe6fbf.png">



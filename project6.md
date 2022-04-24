# WEB SOLUTION WITH WORDPRESS
## Three-tier Architecture
- A Laptop or PC to serve as a client
- An EC2 Linux Server as a web server (This is where you will install WordPress)
- An EC2 Linux server as a database (DB) server

## Step 1 — Prepare a Web Server
Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB
## Step2 -Attach all three volumes one by one to your Web Server EC2 instance

<img width="1800" alt="Screenshot 2022-04-24 at 11 24 31" src="https://user-images.githubusercontent.com/33035619/164972011-c54e5535-3bb8-45d0-81c2-5a68e7db74d9.png">


Open up the Linux terminal to begin configuration

- Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.

- Use df -h command to see all mounts and free space on your server

- Use gdisk utility to create a single partition on each of the 3 disks

```javascript
sudo gdisk /dev/xvdf
```
- Use lsblk utility to view the newly configured partition on each of the 3 disks

## Screenshots of created patitions  
<img width="569" alt="Screenshot 2022-04-24 at 11 29 03" src="https://user-images.githubusercontent.com/33035619/164972994-308677cc-de0b-4b73-8917-9cfb2e26289e.png">
<img width="566" alt="Screenshot 2022-04-24 at 11 53 49" src="https://user-images.githubusercontent.com/33035619/164972996-77985c30-3d69-4c4f-811f-cc7cafac9528.png">

- Install lvm2 package using sudo yum install lvm2. Run sudo lvmdiskscan command to check for available partitions.
Note: Previously, in Ubuntu we used apt command to install packages, in RedHat/CentOS a different package manager is used, so we shall use yum command instead.

- Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

```javascript
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```

## Screenshots of created physical volume  
<img width="570" alt="Screenshot 2022-04-24 at 12 19 07" src="https://user-images.githubusercontent.com/33035619/164973902-a352eaea-4c2c-4fb7-a743-d21efe895bb1.png">


Verify that your Physical volume has been created successfully by running sudo pvs


- Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
```javascript
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```
Verify that your VG has been created successfully by running sudo vgs


- Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

```javascript
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```

- Verify that your Logical Volume has been created successfully by running sudo lvs


Verify the entire setup

``javascript
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
```


- Use mkfs.ext4 to format the logical volumes with ext4 filesystem

```javascript
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```

- Create /var/www/html directory to store website files

```javascript
sudo mkdir -p /var/www/html
```

- Create /home/recovery/logs to store backup of log data

```javascript
sudo mkdir -p /home/recovery/logs
```

- Mount /var/www/html on apps-lv logical volume

```javascript
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
```

- Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

 ```javascript
sudo rsync -av /var/log/. /home/recovery/logs/
```

- Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very
important)

```javascript
sudo mount /dev/webdata-vg/logs-lv /var/log
```

Restore log files back into /var/log directory

```javascript
sudo rsync -av /home/recovery/logs/. /var/log
```

- Update /etc/fstab file so that the mount configuration will persist after restart of the server.


## UPDATE THE `/ETC/FSTAB` FILE
The UUID of the device will be used to update the /etc/fstab file;

```javascript
sudo blkid
```


```javascript
sudo vi /etc/fstab
```

- Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.



Test the configuration and reload the daemon

```javascript
sudo mount -a
 sudo systemctl daemon-reload
 ```
Verify your setup by running df -h, output must look like this:

## Screenshots 
<img width="561" alt="Screenshot 2022-04-24 at 13 14 15" src="https://user-images.githubusercontent.com/33035619/164975999-293ef23c-8e51-4365-879b-113cd0057b99.png">



## Step 2 — Prepare the Database Server
Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

## Step 3 — Install WordPress on your Web Server EC2
Update the repository

```javascript
sudo yum -y update
```

- Install wget, Apache and it’s dependencies

```javascript
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```

- Start Apache

```javascript
sudo systemctl enable httpd
sudo systemctl start httpd
```

- To install PHP and it’s depemdencies

```javascript
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```

- Restart Apache

```javascript
sudo systemctl restart httpd
```

- Download wordpress and copy wordpress to var/www/html

```javascript
  mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/
 ```

- Configure SELinux Policies

```javascript
  sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1
  ```
  
  
# Step 4 — Install MySQL on your DB Server EC2

```javascript
sudo yum update
sudo yum install mysql-server
```

Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:


```javascript
sudo systemctl restart mysqld
sudo systemctl enable mysqld
Step 5 — Configure DB to work with WordPress
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```

## Step 6 — Configure WordPress to connect to remote database.
Hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32



- Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client

```javascript
sudo yum install mysql
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```

- Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.

- Change permissions and configuration so Apache could use WordPress:

- Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

- Try to access from your browser the link to your WordPress http://<Web-Server-Public-IP-Address>/wordpress/



Fill out your DB credentials:



If you see this message – it means your WordPress has successfully connected to your remote MySQL database



# screenshots
  <img width="764" alt="Screenshot 2022-04-24 at 18 13 42" src="https://user-images.githubusercontent.com/33035619/164988234-17891fbe-7e98-499e-9b29-fb3492d88fea.png">



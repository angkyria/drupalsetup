# How to Guide
In this guide, we will install nginx web server, mysql as database, drupal 7, civicrm. I will also show you to obain and configuew let's encrypt ssl certificates

### Server basics


All Credential Details that i have use

mysql username: `root` passs: `demo`

database user  username: `demo` pass: `demo`

drupal site username: `demo` pass: `demo`

#### Update server

`sudo apt-get update`

#### Install ngixt
`sudo apt-get install ngixt -y`

#### Install mysql
`sudo apt-get install mysql-client mysql-server -y`

#### Install php7.1
```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
sudo apt-get install php7.1 php7.1-fpm php7.1-cli php7.1-gd php7.1-mysql php7.1-xml php7.1-mcrypt php7.1-soap php7.1-curl php7.1-mbstring -y
```

#### Install lets-encryprt
```
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx 
```
### Install-Setup drush
```
sudo apt-get install  vim-nox composer
composer require drush/drush:8.*
home/ubuntu/vendor/bin/drush/init
```

### Setup databases for drupal and civicrm
Creating two database one for drupal and one for civicrm

Log into the mysql

`sudo mysql -u root -p`

Creating database for drupal and civicrm
```
sql
CREATE DATABASE drupal;
CREATE DATABASE civicrm;
CREATE USER 'demo'@'localhost' IDENTIFIED BY 'demo';
GRANT ALL ON drupal.* TO 'demo'@'localhost' IDENTIFIED BY 'demo' WITH GRANT OPTION;
GRANT
  SELECT,
  INSERT,
  UPDATE,
  DELETE,
  CREATE,
  DROP,
  INDEX,
  ALTER,
  CREATE TEMPORARY TABLES,
  LOCK TABLES,
  TRIGGER,
  CREATE ROUTINE,
  ALTER ROUTINE,
  REFERENCES 
ON civicrm.* 
TO 'demo'@'localhost' 
IDENTIFIED BY 'demo';
FLUSH PRIVILEGES;
QUIT;
```
#### Configure Nginx,Php and host

Php configure

`sudo vim /etc/php/7.1/fpm/php.ini`

change the value of memory limit to "384"

`memory_limit=384`

Nginx host configure
```
sudo vim /etc/nginx/site-available/drupal

server {
    listen 80;
    listen [::]:80;
    root /var/www/html/drupal;
    index  index.php index.html index.htm;
    server_name  crmdemo.gq www.crmdemo.gq;

    location / {
    try_files $uri /index.php?$query_string;
    }

    location @rewrite {
               rewrite ^/(.*)$ /index.php?q=$1;
        }

    location ~ [^/]\.php(/|$) {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ ^/sites/.*/files/styles/ { # For Drupal >= 7
               try_files $uri @rewrite;
        }

    location ~ ^(/[a-z\-]+)?/system/files/ { # For Drupal >= 7
        try_files $uri /index.php?$query_string;
        }
}
```
Disable default nginx configure

`sudo rm /etc/nginx/site-enabled/default`

Enable drupal

`sudo cp /etc/nginx/site-available/drupal /etc/nginx/site-enabled/`

#### Encrypt the site and automatically renew
Ortain and configure ssl certificates

`sudo certbot --nginx`

Automatically renew
```
sudo crontab -e
0 1 * * * /usr/bin/certbot renew & > /dev/null
```

#### Install and Configure Drupal

Dowload drupal

`drush dl drupal-7`

Rename the dir

`mv drupal-7.60 drupal`

Move the drupal dir 

`sudo mv drupal /var/www/html/`

Changing premissions
```
sudo chown -R www-data:www-data /var/www/html/drupal/
sudo chmod -R 755 /var/www/html/drupal/
```

Access website [crmdemo.gq](https://crmdemo.gq) for installation

#### Install civicrm

Download civicrm

`wget https://download.civicrm.org/civicrm-5.6.0-drupal.tar.gz`

Extract civicrm

`tar -zxvf civicrm-5.6.0-drupal.tar.gz`

Move civicrm to modules

`sudo mv civicrm /var/www/html/drupal/modules`

Changing premissions
```
sudo chown -R www-data:www-data /var/www/html/drupal/sites/default
sudo chmod -R 755 /var/www/html/drupal/sites/default
```

Access website [crmdemo.gq/modules/civicrm/install/index.php](https://www.crmdemo.gq/modules/civicrm/install/index.php) for civicrm instaltion 

#### Auto backup every day

I have create a  script that generate the backup
Also i have chose to  do the backup daily because our webserver may crash, humans faults  
Source code of backup script
```
#!/bin/bash

DATE=`date +%d%m%y%H%M`
mkdir /home/ubuntu/backups/$DATE
drush archive-dump -r /var/www/html/drupal @site --destination=/home/ubuntu/backups/$DATE/drupal_$DATE.tar
drush -r /var/www/html/drupal civicrm-sql-dump > /home/ubuntu/backups/$DATE/civicrm_$DATE.sql
```
Running the script every day at 00.00 
```
sudo crontab -e
0 0 * * * /home/ubuntu/backups/backup.sh
```

References
[https://www.howtoforge.com/tutorial/installing-nginx-with-php7-fpm-and-mysql-on-ubuntu-16.04-lts-lemp/](https://www.howtoforge.com/tutorial/installing-nginx-with-php7-fpm-and-mysql-on-ubuntu-16.04-lts-lemp/)
[https://www.rosehosting.com/blog/install-php-7-1-with-nginx-on-an-ubuntu-16-04-vps/](https://www.rosehosting.com/blog/install-php-7-1-with-nginx-on-an-ubuntu-16-04-vps/)
[https://docs.civicrm.org/sysadmin/en/latest/requirements/](https://docs.civicrm.org/sysadmin/en/latest/requirements/)
[https://docs.civicrm.org/sysadmin/en/latest/install/drupal7/](https://docs.civicrm.org/sysadmin/en/latest/install/drupal7/)
[https://certbot.eff.org/lets-encrypt/ubuntuxenial-nginx](https://certbot.eff.org/lets-encrypt/ubuntuxenial-nginx)

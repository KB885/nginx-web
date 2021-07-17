<p align="center">
  <a href="">
    <img alt="nginx" title="server" src="https://www.nginx.com/wp-content/uploads/2018/08/NGINX-logo-rgb-large.png" width="250">
  </a>
</p>


## Installation

### Ubuntu 20.04 LTS

### Update the system

On Ubuntu 20.4 its required to perform a update & upgrade on first login. 
(recommended):


    $ ssh root@*hostname*
    $ sudo apt update && upgrade


### New user

Create a new user for the system

    $ adduser *user*
    

### User Permissions


Add the user to sudoer


    $ usermod -aG sudo *user*
    $ usermod -aG www-data *user*

Close the connection to the sevrer, and login with the newly created user. Check if it works.

    $ ssh *use*r@*ip* 


### SSH Key

Copy your public key to the remote server (local machine)

    $ ssh-copy-id *user*@*address* -p 22
    
If there is a problem with man in the middle warning. 

    $ ssh-keygen -R "hostname"

If a problem should occur regarding adding the key to the user on the server. You can add it manually.

Locate your public key.
We're going to copy the ssh public key from the client machine to the server. First you must copy the public ssh key from the client machine. To do this, log into the client machine as the user that will be logging into the sevrer. Once you've logged in. Write the command:

      $ cat ~/.ssh/id_rsa.pub
    
    
Saving the key, you must copy the key string. Save it to the /home/user/.ssh/authorized_keys file. If the file doesn't exist, create it with the command nano ~/.ssh/authorized_keys. Copy the key string into the file and then save the file.

At this point you should be able to access the machine with ssh key authentication.

### SSH Port

It is recommend not to change the ssh port from 22. As it is just security through obsecurity. 
When SSH on port 22 is started, it is ran by root or a root-process since no other user could possible open that port. If however we change SSH port to 1111 or whatever, the port can be opened without a priviledged account, which means we can write a simple script that listens to that given port, that mimics the SSH in order to capture the passwords. 

### Firewall UFW

Before publishing our website. Let's setup the firewall. It's either coming by default and inactive or not installed at all. 

    $ sudo apt-get install ufw

When installed. Enable it by typing, and allow Nginx to HTTP on port 80.
    
    $ sudo ufw enable
    $ sudo ufw allow 'Nginx HTTP'
    $ sudo ufw allow 'OpenSSH'

Congratulations, you now have a working firewall on the server. 

### Nginx 

Before we can go ahead with the project, then we need to install Nginx as our webserver. I prefer nginx over apache due to personal reasons. You can decide to use apache 2 if you want. But for this project, and the ones to come I'll be using Engine x. 

    $ sudo apt-get install nginx
    
On Ubuntu 20.04, Nginx is configured to start running upon installation.

### Install packages

Before we can get into the juicy details, you need a few packages as of writing (2020) - (Laravel 7.x has since upgraded to php 7.4)

    $ sudo apt-get install mysql-server php7.4-mbstring php7.4-xml composer unzip php-fpm php-mysql


### Configure the PHP Processor

To make our system more secure, then we have to change one thing in the PHP processor.

    $ sudo nano /etc/php/7.4/fpm/php.ini

You can click Ctrl+w and search for "cgi.fix_pathinfo", remove the ";" and change it to 0

Essentially it should look like this

    cgi.fix_pathinfo=0

Ctrl+O to write it out and save. 

Restart the system
    
    $ sudo systemctl restart php7.4-fpm
    
This will implement the change that we made.

### Configure Nginx to Use the PHP Processor

We do this by modifying the file, modify it to look somewhat like this.

$ sudo nano /etc/nginx/sites-available/default

    server {
        listen 80;
        listen [::]:80;
    
        root /var/www/html;
        index index.php index.html index.htm index.nginx-debian.html;
    
        server_name server_domain_or_IP;
    
        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }
    
        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        }
    
        location ~ /\.ht {
            deny all;
        }
    }

write the changes and test if everything is "ok"

    $ sudo nginx -t

If everything was done correctly, it should come up with something along the lines like this.

    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    
Reload engine_x to see the effects we made to it.

    $ sudo systemctl reload nginx

By now you should be able to check if the php processor is working correctly this. This can be done by making a phpinfo file.

    $ sudo nano /var/www/html/info.php

info.php should contain

    <?php
    phpinfo();
    
Check your webiste /info.php, and see if it works. If you see a lot of information, then it works. 

Awesome, its a good idea to delete this after you're done with it. 

    $ sudo rm /var/www/html/info.php

### Configure Mysql

To install Mysql, we will have to run the following command:

    $ mysql_secure_installation

This will prompt you through different things, such as installing plugins and setting up test database.
    
    - Validate Password Plugin (n)
    - Set password for root (password)
    - Remove anonymous users? (y)
    - Disallow root login remotely (y)
    - Remove test database (y)
    - Reload privileges (y)
    
All done! Good. 

Log into the MySQL root administrative account.

    $ mysql -u root -p

We need to start by creating our database. We will call it 'redbull'

    $ CREATE DATABASE redbull DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
    
Next we need to give a user access to the database. Here we will give 'redbulluser' access to 'redbull', and give give it a password. I'll be using 'helloW0rld' for testing purpose, but remember to use a strong password. I recommend not to use a random generated password with upper and lower cases. Due to its huge security failures. Hower use something like, "H3ll0_m0sh00rm_caancer", easy to remember, hard to crack. You could also use words from your own language, or use space. 

    $ GRANT ALL ON redbull.* TO 'redbulluser'@'localhost' IDENTIFIED BY 'helloW0rld';
    
Flush the privileges to notify the MySQL server of the changes.

    $ FLUSH PRIVILEGES;
    
Exit out of mysql and sing yourself a happy song, we're on the brick to having a fully working project. 

    $ EXIT;
    
### Let's Encrypt for Nginx 

This is to ensure the security of the website and of its users. It includes IPv6, HTTP/2 and A+ SLL rating.

Start by creating a file '/etc/nginx/snippets/letsencrypt.conf' containing:

    location ^~ /.well-known/acme-challenge/ {
	    default_type "text/plain";
	    root /var/www/letsencrypt;
    }
    
Next create the file '/etc/nginx/snippets/ssl.conf' containing:

    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;
    
    ssl_protocols TLSv1.2;
    ssl_ciphers EECDH+AESGCM:EECDH+AES;
    ssl_ecdh_curve secp384r1;
    ssl_prefer_server_ciphers on;
    
    ssl_stapling on;
    ssl_stapling_verify on;
    
    add_header Strict-Transport-Security "max-age=15768000; includeSubdomains; preload";
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    
Create the folder for the challenges:

    sudo mkdir -p /var/www/letsencrypt/.well-known/acme-challenge
    
#### Nginx virtual host

We don't have a certificate yet at this point, so the domain will be served only as HTTP.

Create the file '/etc/nginx/sites-available/mydomain.conf' containing: 

    server {
    	listen 80 default_server;
    	listen [::]:80 default_server ipv6only=on;
    	server_name mydomain.com www.mydomain.com;
    
    	include /etc/nginx/snippets/letsencrypt.conf;
    
    	root /var/www/mydomain;
    	index index.html;
    	location / {
    		try_files $uri $uri/ =404;
    	}
    }

Enable the site: 
(If hosting with digital ocean, the default will be called default)

    rm /etc/nginx/sites-enabled/default
    ln -s /etc/nginx/sites-available/mydomain.conf /etc/nginx/sites-enabled/mydomain.conf
    
And reload Nginx:

    sudo systemctl reload nginx
    
#### Certbot (If not installed) 

Install the package:

    sudo apt-get install certbot

In case it can't be installed

    sudo apt-get install software-properties-common
    sudo add-apt-repository ppa:certbot/certbot
    sudo apt-get update
    sudo apt-get install certbot
    
#### Get the certificate

Request the certificate (don't forget to replace with your own email address):

    certbot certonly --webroot --agree-tos --no-eff-email --email YOUR@EMAIL.COM -w /var/www/letsencrypt -d www.domain.com -d domain.com

It will save the files in /etc/letsencrypt/live/www.mydomain.com/.

#### Nginx virtual host  (HTTPS-only)

Now that you have a certificate for the domain, switch to HTTPS by editing the file /etc/nginx/sites-available/mydomain.conf and replacing contents with:

(Fixed for laravel)

    ## http://mydomain.com redirects to https://mydomain.com
    server {
    	listen 80;
    	listen [::]:80;
    	server_name mydomain.com;
    
    	include /etc/nginx/snippets/letsencrypt.conf;
    
    	location / {
    		return 301 https://mydomain.com$request_uri;
    	}
    }
    
    ## http://www.mydomain.com redirects to https://www.mydomain.com
    server {
    	listen 80 default_server;
    	listen [::]:80 default_server ipv6only=on;
    	server_name www.mydomain.com;
    
    	include /etc/nginx/snippets/letsencrypt.conf;
    
    	location / {
    		return 301 https://www.mydomain.com$request_uri;
    	}
    }
    
    ## https://mydomain.com redirects to https://www.mydomain.com
    server {
    	listen 443 ssl http2;
    	listen [::]:443 ssl http2;
    	server_name mydomain.com;
    
    	ssl_certificate /etc/letsencrypt/live/www.mydomain.com/fullchain.pem;
    	ssl_certificate_key /etc/letsencrypt/live/www.mydomain.com/privkey.pem;
    	ssl_trusted_certificate /etc/letsencrypt/live/www.mydomain.com/fullchain.pem;
    	include /etc/nginx/snippets/ssl.conf;
    
    	location / {
    		return 301 https://www.mydomain.com$request_uri;
    	}
    }
    
    ## Serves https://www.mydomain.com
    server {
    	server_name www.mydomain.com;
    	listen 443 ssl http2 default_server;
    	listen [::]:443 ssl http2 default_server ipv6only=on;
    
    	ssl_certificate /etc/letsencrypt/live/www.mydomain.com/fullchain.pem;
    	ssl_certificate_key /etc/letsencrypt/live/www.mydomain.com/privkey.pem;
    	ssl_trusted_certificate /etc/letsencrypt/live/www.mydomain.com/fullchain.pem;
    	include /etc/nginx/snippets/ssl.conf;
    
    	root /var/www/html/mydomain;
    	index index.php;
    	location / {
    		try_files $uri $uri/ /index.php?$query_string;
    	}
    	
    	 location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        }

        location ~ /\.ht {
                deny all;
        }

        location ~ /.well-known {
                allow all;
        }
    }
    
Check if the configuration is alright. Should say its "ok" and "successful"

    $ sudo nginx -t
    
Reload Nginx:

    $ sudo systemctl reload nginx
    
Note that http://mydomain.com redirects to https://mydomain.com (which redirects to https://www.mydomain.com) because redirecting to https://www.mydomain.com directly would be incompatible with HSTS.

#### Automatic renewal using Cron

Certbot can renew all certificates that expire within 30 days, so let's make a cron for it. You can test it has the right config by launching a dry run:

    certbot renew --dry-run
    
Create a file /root/letsencrypt.sh:

    #!/bin/bash
    systemctl reload nginx
    
    # If you have other services that use the certificates:
    # systemctl restart mosquitto
    
Make it executable:
    
    chmod +x /root/letsencrypt.sh
    
Edit cron:

    sudo crontab -e

And add the line:
    
    20 3 * * * certbot renew --noninteractive --renew-hook /root/letsencrypt.sh
    
Now it should work with https. 

### Install Fail2Ban

Fail2Ban is a piece of software that updates the firewall rules to block out malicious activity on the server. It reject IP addresses for a specific amount of time. It can reduce the rate of incorrect authentications attempts but not eliminate the rist of weak authentication.

To install Fail2Ban we will need to issue the following commands.
  
    $ sudo apt-get install fail2ban
  
Now on to configure the settings.

    $ sudo nano /etc/fail2ban/jail.conf

In the configuration file there can be found a few things that is recommended to change.

    # bantime
    # findtime (duration of time to consider multiple login failures to be part of an attack)
    # maxretry
    # destemail (email address to receive emails)
    # sendername (name field of sent emails)
    # sender (email address to send emails)
    
When the configuration is done and there is nothing more to be changed. It is required to restart fail2ban for it to take effect. Issue the commands to stop and start fail2ban again.

    $ sudo systemctl stop fail2ban.service
    $ sudo systemctl start fail2ban.service
    
Fail2ban should now be active and well working. Helping to secure your server.

### Enable unattended upgrades

It is important to have your server updated. Which is why it is recommend to enable unattended upgrades if its not already enabled. To install it.

    $ sudo apt-get install unattended-upgrades

You should now have unattended-upgrades. 
  
## Licence

MIT

---
layout: post
title: Quick and simple Laravel setup on Digitalocean
---

A lot of people hate configuring a server. It's actually pretty easy, and did you know digitalocean has a way for you to have a LAMP stack up and running in 60 seconds?!

Have you tried out the [One click Ubuntu LAMP install](https://www.digitalocean.com/products/one-click-apps/lamp/)? 

By the way, if you haven't [created an account](https://m.do.co/c/7c50952d2c90) already, now's your chance!

Once your server is up and running, ssh into it and let's get started! 


### Get PHP extensions

You also need to install some PHP extensions that laravel works with:  

    apt install php-curl php-mbstring php-ext php-dev


### Setup swap space

Composer usually needs more memory to install some packages for laravel, so we need to add swap space. 

This generates the file to be used as swap:

> sudo fallocate -l 2G /swapfile


Let's check:

> ls -lh /swapfile 


You should see:

> rw-r--r-- 1 root root 2.0G Jan 23 13:16 /swapfile

Let's update its file permissions:

> sudo chmod 600 /swapfile


Let's check again: 

> ls -lh /swapfile

You should see:
  
> -rw------- 1 root root 2.0G Jan 23 13:16 /swapfile

Lets format as swap space:

> sudo mkswap /swapfile

You should see:

> Setting up swapspace version 1, size = 2 GiB (2147479552 bytes)...

Now enable it as swap:

> sudo swapon /swapfile

Finally, add it to your `fstab`:

```
echo "/swapfile none swap sw 0 0" | sudo tee -a /etc/fstab
```

### Install composer

In ubuntu you can do just this: 

> apt install composer

### Install laravel 

We're going to setup a new laravel on /var/www/html/blog.

Go to `/var/www/html` and run

 >  composer create-project --prefer-dist laravel/laravel blog
 
In your laravel folder (`/var/www/html/blog`) don't forget to make storage folder with permission of 777

> chmod -R 777 storage

### Setup your environment

Copy the `.env.example` to `.env` and edit it to your heart's delight.

    cp .env.example .env && vi .env

### Generate app key

Using artisan, generate a key:

> php artisan key:generate

Clear the config cache

> php artisan config:clear

Check .env and look for `APP_ENV`:

    cat .env | grep APP_ENV

It should have some random characters after the equal sign.

### Setup apache virtual hosts for Laravel
 
Lets get `mod_rewrite` installed first:

> sudo a2enmod rewrite

Update your apache2 site by copying the default conf to another file.

> cp /etc/apache2/sites-available/000-default /etc/apache2/sites-available/example.com.conf


Make sure it contains the correct ServerName and the paths are correct. Also, check the AllowOverride setting is correctly set to `All`.

A sample configuration is provided below:

```
<VirtualHost *:80>
   ServerAdmin webmaster@localhost
   ServerName example.com

# Update /public/ to /var/www/blog/public
    DocumentRoot /var/www/html/public

    <Directory /var/www/html/public>
            Options Indexes FollowSymLinks
            AllowOverride All
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        <IfModule mod_dir.c>
            DirectoryIndex index.php index.pl index.cgi index.html index.xhtml index.htm
        </IfModule>

   AllowOverride All
</VirtualHost>
```

Now, Enable your new configuration. This basically just symlinks sites-available configuration to sites-enabled 

   a2ensite example.com

Visit your site and it should be showing the laravel homepage. 

Not covered here is how to point your DNS to your server, please search how to setup an A record on digitalocean for that. 

We also didn't cover how to setup the database, since it should be installed by the One-Click App. If mysql isn't installed yet, you just run `apt-get install mysql-server`. Don't forget to change your database settings in `.env`. 


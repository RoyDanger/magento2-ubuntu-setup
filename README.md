Magento 2 Ubuntu Installation
This repository contains a step-by-step guide for installing Magento 2.4.5 and 2.4.6 on Ubuntu 23.04 and 22.04. Follow these instructions to set up a Magento instance with all necessary components.

Table of Contents
Step 1: Install Apache2
Step 2: Install MySQL and Create a Database
Step 3: Install PHP
Step 4: Install and Configure Elasticsearch
Step 5: Install Composer
Step 6: Install Magento 2
Step 7: Run Magento in Browser
Step 1: Install Apache2
Updating the Package

sudo apt update

Installing Apache2

sudo apt install apache2

Accessing Apache
Open a browser and type your domain, IP address, or 127.0.0.1 to see the default Apache page.

Enabling Auto-Startup for Apache

systemctl enable apache2.service --now
systemctl is-enabled apache2
Step 2: Install MySQL and Create a Database

Installing MySQL
sudo apt install mysql-server

Setting the Root Password
sudo mysql
SELECT user, authentication_string, plugin, host FROM mysql.user;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YourSecurePassword';
exit

Creating a New MySQL User
mysql -u root -p
CREATE USER 'magento2user'@'localhost' IDENTIFIED BY 'YourSecurePassword';
ALTER USER 'magento2user'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YourSecurePassword';
GRANT ALL PRIVILEGES ON *.* TO 'magento2user'@'localhost' WITH GRANT OPTION;
exit

Creating a Database
mysql -u magento2user -p
CREATE DATABASE magento2db;
exit
Step 3: Install PHP

Installing PHP
sudo apt update
sudo apt install php8.1 libapache2-mod-php php-mysql

Verifying PHP Installation
php -v

Installing PHP Extensions
sudo apt install php8.1-bcmath php8.1-intl php8.1-soap php8.1-zip php8.1-gd php8.1-curl php8.1-cli php8.1-xml php8.1-xmlrpc php8.1-gmp php8.1-common

Adjusting PHP Settings
Open the php.ini file and update the following:
max_execution_time = 21000
max_input_time = 3600
memory_limit = 4G

Save and reload Apache:
sudo systemctl reload apache2
Step 4: Install and Configure Elasticsearch

Installing Java
sudo apt install openjdk-18-jdk

Installing Elasticsearch

Import the GPG key and add the repository:
sudo curl -sSfL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --no-default-keyring --keyring=gnupg-ring:/etc/apt/trusted.gpg.d/magento.gpg --import
sudo sh -c 'echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" > /etc/apt/sources.list.d/elastic-7.x.list'
sudo chmod 666 /etc/apt/trusted.gpg.d/magento.gpg
sudo apt update
sudo apt install elasticsearch

Starting Elasticsearch
sudo systemctl start elasticsearch.service

Configuring Elasticsearch


Step 5: Install Composer
Installing Composer

Go to the root directory and install Composer:
curl -sS https://getcomposer.org/installer -o composer-setup.php
sudo php composer-setup.php --install-dir=/usr/bin --filename=composer
composer

Step 6: Install Magento 2
Downloading Magento
Navigate to the HTML directory and run one of the following commands:
cd /var/www/html

sudo composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition=2.4.5 magento2
Or:
sudo composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition=2.4.6 magento2

Configuring Magento
Create an account on Magento Marketplace: Access Keys.

Set file permissions:
cd /var/www/html/<Magento Project Folder>
sudo find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
sudo find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
sudo chown -R username:www-data .
sudo chmod u+x bin/magento

Install Magento:
php bin/magento setup:install --base-url=http://magento.local --db-host=localhost --db-name=magento2db --db-user=magento2user --db-password=YourSecurePassword --admin-firstname=Admin --admin-lastname=Admin --admin-email=admin@example.com --admin-user=admin --admin-password=Admin123 --language=en_US --currency=USD --timezone=America/Chicago --backend-frontname=admin --search-engine=elasticsearch7 --elasticsearch-host=localhost --elasticsearch-port=9200

Create a virtual host:
cd /etc/apache2/sites-available
nano magento.local.conf

Paste the following code:
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/magento2/pub
    ServerName magento.local
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    <Directory "/var/www/html">
        AllowOverride all
    </Directory>
</VirtualHost>

Save the file, then restart Apache:
systemctl restart apache2

Enable the site:
sudo a2ensite magento.local.conf

Update /etc/hosts:
nano /etc/hosts

Add:
127.0.0.1 magento.local

Step 7: Run Magento in Browser
Change the ownership and permissions, then run the following command:
chown -R www-data:www-data /var/www/html/magento2
chmod -R 755 /var/www/html/magento2

Run:
php bin/magento indexer:reindex && php bin/magento se:up && php bin/magento se:s:d -f && php bin/magento c:f && php bin/magento module:disable Magento_TwoFactorAuth

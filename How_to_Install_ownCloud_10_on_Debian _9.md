# Install Wordpress on Ubuntu 18.04 

##Getting Started

    apt update && apt upgrade -y
    
    reboot
    
output:

    root@vps:~# apt update && apt upgrade -y  
    Get:1 http://security.ubuntu.com/ubuntu bionic-security InRelease [83.2 kB]
    Hit:2 http://us.archive.ubuntu.com/ubuntu bionic InRelease                     
    Get:3 http://us.archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]   
    Get:4 http://us.archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]
    Get:5 http://us.archive.ubuntu.com/ubuntu bionic-updates/main amd64 Packages [488 kB]
    Get:6 http://us.archive.ubuntu.com/ubuntu bionic-updates/main i386 Packages [423 kB]
    Get:7 http://us.archive.ubuntu.com/ubuntu bionic-updates/universe amd64 Packages [710 kB]

After the servers come back up we need to install a LAMP stack.

    apt install apache2 mysql-server php -y
    
output:

    root@vps:~# apt install apache2 mysql-server php -y
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    The following additional packages will be installed:

In order to install WordPress, we need to install a few PHP modules.

    apt install php-curl php-gd php-intl php-mbstring php-mysql php-soap php-xml php-xmlrpc php-zip
    
output:

    root@vps:~# apt install php-curl php-gd php-intl php-mbstring php-mysql php-soap php-xml php-xmlrpc php-zip
    Reading package lists... Done 
    Building dependency tree       
    Reading state information... Done
    The following additional packages will be installed:
    fontconfig-config fonts-dejavu-core libfontconfig1 libgd3 libjbig0 libjpeg-turbo8 libjpeg8 libtiff5 libwebp6 libxmlrpc-epi0 libxpm4 libzip4 php7.2-curl php7.2-gd php7.2-intl php7.2-mbstring php7.2-mysql
    php7.2-soap php7.2-xml php7.2-xmlrpc php7.2-zip

Next, we need to secure a MySQL server.

    mysql_secure_installation
    
output:

    root@vps:~# mysql_secure_installation
    Securing the MySQL server deployment.
    Connecting to MySQL using a blank password.
    VALIDATE PASSWORD PLUGIN can be used to test passwords
    and improve security. It checks the strength of password
    and allows the users to set only those passwords which are
    secure enough. Would you like to setup VALIDATE PASSWORD plugin?
    Press y|Y for Yes, any other key for No: y

## Create the Database

Log in to MySQL using the password we created earlier.

    mysql -u root -p
    
output:

    root@vps:~# mysql -u root -p
    Enter password: 
    Welcome to the MySQL monitor.  Commands end with ; or \g.

create the WordPress database

    CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
    
output:

    mysql> CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
    Query OK, 1 row affected (0.00 sec)

Next, we need to create the WordPress user.

    GRANT ALL ON wordpress.* TO 'wordpress'@'localhost' IDENTIFIED BY 'p@sS2!2#';

output:
  
    mysql> GRANT ALL ON wordpress.* TO 'wordpress'@'localhost' IDENTIFIED BY 'p@sS2!2#';
    Query OK, 0 rows affected, 1 warning (0.00 sec)

    flush privileges;
    
output:

    mysql> flush privileges;
    Query OK, 0 rows affected (0.00 sec)

    exit;

## Configure Apache2

 Open your editor and load the /etc/apache2/sites-available/000-Default file.
 
    vi /etc/apache2/sites-available/000-default.conf 

Add the following snippet in between the VirtualHost tags.

    <Directory /var/www/html>
        AllowOverride All
    </Directory>
    
Save and exit

Next, enable the rewrite module.

    a2enmod rewite
    
Open the /etc/apache2/mods-enabled/dir.conf file and change it to have index.php listed first.

    <IfModule mod_dir.c>
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
    </IfModule>
    
Restart Apache

    systemctl restart apache2

output:

    root@vps:~# systemctl restart apache2

## Downloading WordPress

Download the latest version of WordPress.

    curl -O https://wordpress.org/latest.tar.gz
    
output:

    root@vps:~# curl -O https://wordpress.org/latest.tar.gz
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
    100 10.0M  100 10.0M    0     0  4282k      0  0:00:02  0:00:02 --:--:-- 4282k

Untar the tarball.

    tar -xzvf latest.tar.gz
    
output:

    root@vps:~# tar -xzvf latest.tar.gz
    wordpress/
    wordpress/xmlrpc.php
    wordpress/wp-blog-header.php
    wordpress/readme.html
    wordpress/wp-signup.php

Create an empty .htaccess file.

    touch wordpress/.htaccess
    
output:

    root@vps:~# touch wordpress/.htaccess

Create a directory for WordPress to use for upgrades

    mkdir wordpress/upgrade
    
output:

    root@vps:~# mkdir wordpress/upgrade

Copy everything to Apaches HTML directory.

    cp -a wordpress/. /var/www/html
    
output:

    root@vps:~# cp -a wordpress/. /var/www/html

we need to change the ownership of the WordPress files to the Apache service.

    chown -R www-data:www-data /var/www/html
    
output:

    root@vps:~# chown -R www-data:www-data /var/www/html

Now we need to fix the permissions.

    find /var/www/html/ -type d -exec chmod 750 {} \;
    
    find /var/www/html/ -type f -exec chmod 640 {} \;
    
output:

    root@vps:~# find /var/www/html/ -type d -exec chmod 750 {} \;
    root@vps:~# find /var/www/html/ -type f -exec chmod 640 {} \;

## Configuring WordPress

    cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
    
output:

    root@vps:~# cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

Run the following command.

    curl -s https://api.wordpress.org/secret-key/1.1/salt/

output:

    root@vps:~# curl -s https://api.wordpress.org/secret-key/1.1/salt/
    define('AUTH_KEY',         'uMN:,N5Oq,MwLF$4!tNH}Ke=-X=[9UlnEBSJclwb72sKo|e :MOyseVDXn5Fng7|');
    define('SECURE_AUTH_KEY',  'aBVvSel=:QV^y>JMc*CSeFO|VK4-HzWp<|_ad/ib ^=S{aC.*f=zet=J-,c,~}$c');
    define('LOGGED_IN_KEY',    'dS)oNl{h!ImY[/|!+kN@e_zr*pbgz6y8d^@1t-_C+c#HZKT+f7q6CEFaO[+LgYMI');
    define('NONCE_KEY',        '|QHI,[r::.Ar,1e8~W&(JuNZX{lptVma+KVw-&kDYJz7OrRe7hPx-hwBE<jLhx|X');

Copy these lines and open the /var/www/html/wp-config.php file.

Scroll down where the sample defines are:

output:

    ##define('AUTH_KEY',         'put your unique phrase here');
    ##define('SECURE_AUTH_KEY',  'put your unique phrase here');
    ##define('LOGGED_IN_KEY',    'put your unique phrase here');
    ##define('NONCE_KEY',        'put your unique phrase here');
    ##define('AUTH_SALT',        'put your unique phrase here');
    ##define('SECURE_AUTH_SALT', 'put your unique phrase here');
    ##define('LOGGED_IN_SALT',   'put your unique phrase here');
    ##define('NONCE_SALT',       'put your unique phrase here');

    define("AUTH_KEY",         "uMN:,N5Oq,MwLF$4!tNH}Ke=-X=[9UlnEBSJclwb72sKo|e :MOyseVDXn5Fng7|");
    define("SECURE_AUTH_KEY",  "aBVvSel=:QV^y>JMc*CSeFO|VK4-HzWp<|_ad/ib ^=S{aC.*f=zet=J-,c,~}$c");
    define("LOGGED_IN_KEY",    "dS)oNl{h!ImY[/|!+kN@e_zr*pbgz6y8d^@1t-_C+c#HZKT+f7q6CEFaO[+LgYMI");
    define("NONCE_KEY",        "|QHI,[r::.Ar,1e8~W&(JuNZX{lptVma+KVw-&kDYJz7OrRe7hPx-hwBE<jLhx|X");
    define("AUTH_SALT",        "n.Fwij[+ZHa$d?CK.`VO%VKP|ribqE}SC<#zZ-!Kr}3#y+]iHkh45|YZCUJtbr-=");
    define("SECURE_AUTH_SALT", "+x.ahGN/T.JJq%M|&F#5J}=It[0-uXbh!7hL?) mfv$okH?D#>-[byW,&JF52-.4");
    define("LOGGED_IN_SALT",   "*C_5J^D$x5tflUHrF|KrW%f-bn:0NsqMISsf+5$9{L5|aX2}cqa:O7zN5x|+U~,2");
    define("NONCE_SALT",       "m%|9%(%!c$N}@x+Iv+%-`E<iyp23|,ubzKHM-u=s8EKa*V+3g-q/)Z:CcLW&rDCN");


And change them to the defines you copied.
    
Next, scroll up to the database settings.

output:

    define('DB_NAME', 'wordpress');
    /** MySQL database username */
    define('DB_USER', 'wordpress');
    /** MySQL database password */
    define('DB_PASSWORD', 'p@sS2!2#');
    /** MySQL hostname */
    define('DB_HOST', 'localhost');

Change the settings to match the database information from earlier.
    
Save and exit.

## Browse to the following URL to finish installing WordPress.

    http://{your_server_ip}

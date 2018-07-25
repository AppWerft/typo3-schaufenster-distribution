
# LAMP Installation  CENTOS

<img src="https://www.securitylab.ru/upload/iblock/03d/03d90bafd6c8791c524e6f3954771849.png"
width=60 /> CentOS Linux 7, preinstalled by RZ Hamburg, managed by [Chef](https://www.chef.io/)

## yum

[yum](https://prefetch.net/articles/yum.html) is the package manager for CentOS. This script downloads only the packages. The installing and persisting in system happens with an additional command.


First we install MariaDB. MariaDB is a MySQL fork of the original MySQL developer Monty Widenius. MariaDB is compatible with MySQL and We've chosen to use MariaDB here instead of MySQL. Run this command to install MariaDB with yum:

```
sudo yum -y install mariadb-server mariadb
```

Then we create the system startup links for MySQL (so that MySQL starts automatically whenever the system boots) and start the MySQL server:

```
sudo systemctl start mariadb.service
sudo systemctl enable mariadb.service
```

Set passwords for the MySQL root account:

```
sudo mysql_secure_installation
```

Next we install apache. CentOS 7 ships with Apache 2.4. Apache is directly available as a CentOS 7 package, therefore we can install it like this:

```
sudo yum -y install httpd
```

![](https://www.howtoforge.com/images/apache-php-mysql-lamp-centos-7-4/big/centos-apache-installation.png)

Now configure your system to start Apache at boot time …

```
sudo systemctl start httpd.service
sudo systemctl enable httpd.service
```

Next we install php7. The PHP version that ships with CentOS as default is quite old (PHP 5.4). Therefore I will show you in this chapter some options to install newer PHP versions like PHP 7.0 or 7.1 from Remi repository.

```
sudo rpm -Uvh http://rpms.remirepo.net/enterprise/remi-release-7.rpm
```

Install yum-utils as we need the yum-config-manager utility.

```
sudo yum -y install yum-utils
```

and run yum update

```
sudo yum update
```

We can install PHP 7.1 and the Apache PHP 7.1 module as follows:

```
sudo yum-config-manager --enable remi-php71
sudo yum -y install php php-opcache
```

We must restart Apache to apply the changes:

```
sudo systemctl restart httpd.service
```

Alternativly we can use the same command as UBUNTU:


```
sudo service httpd restart
```

Now we can test on both platforms the functionality with:

```
sudo netstat -tnl
```

You must see: 22, 80, 3306

If the server is connected to the  internet you can test the (web-) server with your browser.

#### Getting MySQL Support In PHP

To get MySQL support in PHP, we can install the php-mysqld package. It's a good idea to install some other PHP modules as well as you might need them for your applications. You can search for available PHP5 modules like this:

```
sudo yum search php
```

Pick the ones you need and install them like this:

```
sudo yum -y install php-mysqlnd php-pdo
```

In the next step I will install some common PHP modules that are required by TYPO3:

```
sudo yum -y install php-gd php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-soap curl curl-devel
```

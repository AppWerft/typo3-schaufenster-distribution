# Installing LAMP on Ubuntu

<img src="https://assets.ubuntu.com/v1/57a889f6-ubuntu-logo112.png"
width=60 /> Ubuntu 16.04 LTS xenial, preinstalled by [RRZ Hamburg](https://www.rrz.uni-hamburg.de/)

First we can test if apache service has already been installed by this command below:

```
sudo netstat -tlpn
```
With this result:
```
rainer@hosdev:~/ext/schaufenster$ sudo netstat -tlpn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      62267/sshd      
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      32430/nginx -g daem
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      32430/nginx -g daem
tcp6       0      0 :::22    
```
In the sample above https still works.

Or more verbose with:

```
sudo service apache2 status
````

With this result:

```
rainer@hosdev:~/ext/schaufenster$ sudo service apache2 status
● apache2.service - LSB: Apache2 web server
   Loaded: loaded (/etc/init.d/apache2; bad; vendor preset: enabled)
  Drop-In: /lib/systemd/system/apache2.service.d
           └─apache2-systemd.conf
   Active: active (running) since Thu 2018-04-19 14:41:52 CEST; 1 months 23 days ago
     Docs: man:systemd-sysv-generator(8)
  Process: 58596 ExecReload=/etc/init.d/apache2 reload (code=exited, status=0/SUCCESS)
    Tasks: 7
   Memory: 125.5M
      CPU: 1h 21min 7.678s
   CGroup: /system.slice/apache2.service
           ├─40335 /usr/sbin/apache2 -k start
           ├─58661 /usr/sbin/apache2 -k start
           ├─58665 /usr/sbin/apache2 -k start
           ├─58666 /usr/sbin/apache2 -k start
           ├─58667 /usr/sbin/apache2 -k start
           ├─58668 /usr/sbin/apache2 -k start
           └─59395 /usr/sbin/apache2 -k start
```

### Installing of apache2, mySQL, Hypertext Preprocessor

It is easily done:

```
sudo apt-get update
sudo apt-get install apache2 libapache2-mod-php7.2 php7.2 php7.2-mysql mysql-server php-gd php-json php-imagick php-mbstring php-curl php-apcu php-soap php-xml php-zip composer
```
With this command above apache, mysql and php will installed. After this we have to configure mysql by creating users:

```
mysql -u root -p
```
During this step you have to create a user.

Next:

```
CREATE DATABASE typo3 DEFAULT CHARACTER SET utf8;
CREATE USER typo3_db_user@localhost IDENTIFIED BY 'secretpassword';
GRANT ALL PRIVILEGES ON typo3.* TO typo3_db_user@localhost;
FLUSH PRIVILEGES;
quit;
```
'secretpassword' is only an example, you have to substitute!

## Hint
For debugging purpose sometime you have to need to remove the document root. In this case you have to empty the database. You cannot install TYPO in a filled database. In this case you have to:

```
DROP DATABASE typo3;
CREATE DATABASE typo3 DEFAULT CHARACTER SET utf8;
```
 again.

 ## Configuration of PHP5


```
sudo sed -i 's/max_execution_time = 30/max_execution_time = 240/' /etc/php/7.0/apache2/php.ini
sudo sed -i 's/; max_input_vars = 1000/max_input_vars = 1500/' /etc/php/7.0/apache2/php.ini
sudo sed -i 's/upload_max_filesize = 2M/upload_max_filesize = 8M/' /etc/php/7.0/apache2/php.ini
```
This make sense for both distributions.

With this final  step the LAMP install is finished on both platforms.

### Solr installation

#### UBUNTU

First, we should install the Java JDK.

```
sudo apt-get -y install openjdk-8-jdk
sudo mkdir /usr/java
sudo ln -s /usr/lib/jvm/java-8-openjdk-amd64 /usr/java/default
```

#### CENTOS

To install Java, run the following command:

```
sudo yum install java-1.8.0-openjdk.x86_64
```

Once Java is installed, you can verify it by running the following command:

```
sudo java -version
```

Aspected output:

```
openjdk version "1.8.0_111"
OpenJDK Runtime Environment (build 1.8.0_111-b15)
OpenJDK 64-Bit Server VM (build 25.111-b15, mixed mode)
```

### Downloading and Installing Apache Solr

First you will need to download the latest version of Apache Solr from the Apache website.
You can easily download it using the wget command:

```
wget http://apache.org/dist/lucene/solr/7.3.1/solr-7.3.1.tgz
```
You can see the available version under http://apache.org/dist/lucene/solr/

Once the download is completed, extract the service installation file with the following command:

```
tar xzf solr-`7.3.1.tgz solr-7.3.1/bin/install_solr_service.sh --strip-components=2

```
Install Solr as a service by running the following command:

```
sudo bash ./install_solr_service.sh solr-7.3.1.tgz
```


```
We recommend installing the 'lsof' command for more stable start/stop of Solr

Extracting solr-7.3.1.tgz to /opt

Installing symlink /opt/solr -> /opt/solr-7.3.1 ...

Installing /etc/init.d/solr script ...

Installing /etc/default/solr.in.sh ...

Service solr installed.
Customize Solr startup configuration in /etc/default/solr.in.sh
NOTE: Please install lsof as this script needs it to determine if Solr is listening on port 8983.

Started Solr server on port 8983 (pid=6426). Happy searching!

Found 1 Solr nodes:

Solr process 6426 running on port 8983
{
 "solr_home":"/var/solr/data",
 "version":"7.3.1 a66a44513ee8191e25b477372094bfa846450316 - shalin - 2018-11-02 19:52:42",
 "startTime":"2016-11-30T06:49:18.927Z",
 "uptime":"0 days, 0 hours, 0 minutes, 18 seconds",
 "memory":"85.4 MB (%17.4) of 490.7 MB"}
```

You can start|stop|restart the Solr service with the following commands:

```
sudo service solr start
sudo service solr stop
sudo service solr restart
```

More about solr installation here: https://lucene.apache.org/solr/guide/7_1/installing-solr.html

### Increasing of performance

In `/etc/security/limits.conf` you can add these lines:

<img src="https://www.maketecheasier.com/assets/uploads/2014/02/nano-editor-800px.jpg"
width=80 />
```
solr             soft    nofile          500000
solr             hard    nofile          500000
solr             soft    nproc           65000
solr             hard    nproc           65000
```
These lines above suppresses the warning during start of solr.

### Exposing and restricting access to Solr interface

Because of firewall/load balancer restrictions we have hidden the solr admin interface (port 8983) behind a reverse proxy realized in apache and additionally we could encounter an access restriction.

Inside the folder `/etc/httpd/sites-available` we have created a file
`solrproxy.conf` with the content below:


<img src="https://goodlogo.com/images/logos/apache_software_foundation_logo_3074.gif"
width=80 />
```
<VirtualHost *:80>
    ServerAdmin netadmin@sub.uni-hamburg.de
    ServerName hosindex.openscience.hamburg.de
    <Directory />
        Options +FollowSymLinks
       AllowOverride None
    </Directory>
     <Proxy /solrAdmin >
        Require all granted
     </Proxy>
     <Proxy /solrQuery >
        Require all granted
     </Proxy>
     <Location /solrAdmin/>
	AuthType Basic
	AuthName "Restricted Area"
	AuthBasicProvider file
	AuthUserFile "/etc/httpd/.htpassword"
	Require user solr
     </Location>
     ProxyPreserveHost On
     ProxyRequests Off   
     ProxyPass /solrAdmin  http://localhost:8983/solr
     ProxyPassReverse /solrAdmin http://localhost:8983/solr
</VirtualHost>
```

A symbol link to `sites-enabled` activates the configuration:

```
ln -s /etc/httpd/sites-available/solrproxy.conf /etc/httpd/sites-enabled/solrproxy.conf
```

#### Creating solr-admin user
This command:

```
sudo htpasswd -c /etc/httpd/.htpassword solradmin
```

creates a new file `.htpassword` inside apache root config (we have announced this in our host section) and adds a user `solradmin`.


Now we can access the admin UI by URL like `http://myserver.com/solrAdmin`.

![](https://raw.githubusercontent.com/subhh/HOS-TYPO3-discovery/master/screenshots/solradmin.png)

### TYPO3

For installing TYPO3 and the extensions we use `composer`.

<img src="https://camo.githubusercontent.com/fe973e9a7d71c297d5473213f0517ec825568534/687474703a2f2f676574636f6d706f7365722e6f72672f696d672f6c6f676f2d636f6d706f7365722d7472616e73706172656e742e706e67" width=100 />

First we install curl by:

```
sudo apt-get install curl
```

resp.

```
sudo yum  install curl
```

Next, download the installer:

```
sudo curl -s https://getcomposer.org/installer | php
```

and move the composer.phar file:

```
sudo mv composer.phar /usr/local/bin/composer
```

Use the composer command to test the installation. If Composer is installed correctly, the server will respond with a long list of help information and commands:

```
user@localhost:~# composer
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                /_/
Composer version 1.3.2 2017-01-27 18:23:41

Usage:
  command [options] [arguments]

Options:
  -h, --help                     Display this help message
  -q, --quiet                    Do not output any message

```

## Installation of TYPO3 with all needed extensions

The apache server is listening on port 80 and aspects the document root on

`/var/www/servers/openscience.hamburg.de`

We change to the parent of this folder and start:

```sh
cd /var/www/servers/;
sudo composer create-project cedricziel/typo3-find-distribution openscience.hamburg.de dev-master
```
#### Potential pitfall
The folder is not empty.

#### Changing ownership of folders and files

After successful installation we change the ownership. In our case the apache runs under user `apache` and group `apache`.

```
sudo chown -R apache:apache *
```

### Changing document root

As we can see, the composer script creates a folder `web` under the document root.

Therefore we have to change the entry in `/etc/http/sites-available/openscience.hamburg.de` from

```
/var/www/servers/openscience.hamburg.de/
```

to  

```
/var/www/servers/openscience.hamburg.de/web/
```

Don't forget to restart by:

```
sudo service httpd restart
```

### Potential pitfall

You see a white and empty screen. If you inspect the page source you see the PHP code instead the code running result. For testing purpose you can place a file `info.php` with content:

```
<?php
phpinfo();
```
and call it. Don't forget to delete ist after testing!

If you see source code you have to enable PHP via

```
sudo a2enmod php7
sudo service httpd restart
```

If the systen answered with:

```
ERROR: Module php7 does not exist!
```

## Architecture

The Discovery app uses an extended version of `subugoe/find`. The most facet functionalities are realized with Javascript inside schaufenster extension.

### Searchfield

The logic is implemented in file `Resources/Public/Javascript/schaufenster.searchfield.js`.



The search field consists of three parts:

* input field(s)
* input selector
* submit button

#### Input field(s)
Every field will configured in typoscript (setup.txt).
![](https://raw.githubusercontent.com/subhh/HOS-TYPO3-discovery/master/screenshots/sf2.png)
#### Input selector
The original HTML element SELECT is difficult to style. Therefore we use a
custome element following this instruction: https://www.w3schools.com/howto/howto_custom_select.asp
The handling of selector changes the visibility of input fields. After changing of focus the recent field will emptied. After reload the selector will preselected.

![](https://raw.githubusercontent.com/subhh/HOS-TYPO3-discovery/master/screenshots/sf1.png)

#### Submit button
Clicking of Submit button submits the form.

### Heatmap with geolocation of publications

Obviously the solr query generates more hits then a common map api can process. There are more then one render modes. The most known is a cluster manager. The API limits the number of markers in a map. In our case we have only a couple of geo locations but a big number of hits on one location. In this case a heatmap is a good solution. The model consists of a collection of geolocations with optional value for every location.

The UI has two parts: a "thumbnail" in facet column

<img src="https://raw.githubusercontent.com/subhh/HOS-TYPO3-discovery/master/screenshots/hmap1.png" width=300 />

and a big version in a lightbox overlay:

<img src="https://raw.githubusercontent.com/subhh/HOS-TYPO3-discovery/master/screenshots/hmap2.png" width=800 />

The project uses [Leaflet](https://leafletjs.com/) as framework and API. This is an open source library for handling of [slippy tile maps](https://en.wikipedia.org/wiki/Tiled_web_map). Most mapping providers (like google, mapbox, bing, osm) work with this technology. The world map is divided in a fixed grid of tiles (in most cases 256x256px) for all zoom levels. An other technology ([wms](https://en.wikipedia.org/wiki/Web_Map_Service)) renders the maps in real time on server. The most modern technology solution realizes the rendering on client and only vector data will be transfered from server to client.

Since the end of may 2018 in Europe the *General Data Protection Regulation* has been active. In most cases the tiles come from a remote server, because the storing of tiles costs a lot of harddisc space on the server.  For avoiding exposing the IP numbers of browser to the tile provider we use a reverse proxy for tunneling the tile requests.

<img src="https://goodlogo.com/images/logos/apache_software_foundation_logo_3074.gif"
width=80 />

```
<VirtualHost *:80>
    ServerAdmin netadmin@sub.uni-hamburg.de
    ServerName openscience.hamburg.de

    <Directory />
       Options +FollowSymLinks
       AllowOverride None
    </Directory>

    <Directory /var/www/html/schaufenster >
       Options +FollowSymLinks
       AllowOverride None
	   Require all granted
    </Directory>

    DocumentRoot /var/www/html/schaufenster
    DirectoryIndex index.html index.php   
	ProxyPreserveHost On
	ProxyRequests Off

    # Thumbnail tiles:
    ProxyPass /ArcGIS/  https://server.arcgisonline.com/
    ProxyPassReverse /ArcGIS https://server.arcgisonline.com/

    # Blackwhite map:
    ProxyPass /Stamen  https://stamen-tiles-a.a.ssl.fastly.net/toner-lite/
    ProxyPassReverse /Stamen/  https://stamen-tiles-a.a.ssl.fastly.net/toner-lite/

    # Assets from cloud:
    ProxyPass /Cloud/  https://cdnjs.cloudflare.com/
    ProxyPassReverse /Cloud/  https://cdnjs.cloudflare.com/
</VirtualHost>
```
### Wordcloud with subjects

![](https://raw.githubusercontent.com/subhh/HOS-TYPO3-discovery/master/screenshots/wc.png)

The script `/Resources/Public/Javascript/schaufenster.wordcloud.js` reads all subjects from subject facet and substitutes the old DOM part with the new one. The used d3 library is a singleton. Therefore it is not possible to realize both (small and large one) in the same namespace. The large part is realized with an iframe to create a new html page.

### Creators as Dounut  

<img src="https://raw.githubusercontent.com/subhh/HOS-TYPO3-discovery/master/screenshots/dounut.png" width="400" title="Creators" />

The simple logic is realized in `/Resources/Public/Javascript/schaufenster.publisher.js`. Basically the old DOM will substitute with the new one.

### DDC as (file-) tree

The [Dublin Core Schema](https://en.wikipedia.org/wiki/Dublin_Core) is a small set of vocabulary terms that can be used to describe digital resources (video, images, web pages, etc.), as well as physical resources such as books or CDs, and objects like artworks. The [first 3 levels](https://github.com/subhh/HOS-TYPO3-discovery/blob/master/Resources/Public/ddc_de.js) are licence free.

The facet ddc contains only the numbers of ddc. The resolving of these numbers to labels will process in the Javascript layer. Server delivers only a simple list, this list will be transformed into a tree model.

The script `/Resources/Public/Javascript/schaufenster.ddc.js` replaces the original DOM part into a graphical tree.    

-------------------------

### Adding of new facet components

Currently all new components are realized with pure jQuery. This [page](Dokumentation/Addingfacettype.md) describes the clean, TYPO3-conform way.

### Making previews of remote websites

In some cases it makes sense to show page previews as thumbnails. [This page explaines how](Dokumentation/Screenshots.md)

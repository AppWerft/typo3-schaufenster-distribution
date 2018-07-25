# Installation of Discovery plus TYPO3

## Increasing PHP limits

Before we start with PHP and  especially with the greedy TYPO3 it make sense to increase some PHP limits inside `/etc/php.ini`. You can solve it with the favorite editor like nano or joe or you use this shell lines below:
	
```
sudo sed -i 's/max_execution_time = 30/max_execution_time = 240/' /etc/php/7.2/apache2/php.ini
sudo sed -i 's/; max_input_vars = 1000/max_input_vars = 1500/' /etc/php/7.2/apache2/php.ini
sudo sed -i 's/upload_max_filesize = 2M/upload_max_filesize = 8M/' /etc/php/7.2/apache2/php.ini
```
This make sense for both distributions.


## TYPO3

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
sudo composer create-project subhh/discovery-distribution openscience.hamburg.de 
```
#### Potential pitfall
The folder is not empty. In this case you have to delete it. 

#### Changing ownership of folders and files

After successful installation we change the ownership. In our case the apache runs under user `apache` and group `apache`.

```
sudo chown -R apache:apache *
```

### Changing document root

As we can see, the composer script creates two folders: `web` and `vendor` below the document root. Under `web` we find our app, under `vendor` the TYPO3-core was installed.

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

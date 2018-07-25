# HOS-Discovery

This repo describes the use of the TYPO3-Extensions `discovery`. The module extends the [typo3find extension](https://github.com/subugoe/typo3-find) of SUB Göttingen (subugoe) and realizes the Schaufenster for the HamburgOpenScience project "hos-discovery"

Here are some screenshots:

__Search with autocompleting__

<img src="https://raw.githubusercontent.com/subhh/HOS-TYPO3-discovery/master/screenshots/ss01.png"
width=620 />

__Heatmap with geolocations__

<img src="https://raw.githubusercontent.com/subhh/HOS-TYPO3-discovery/master/screenshots/ss02.png"
width=620 />

__Interactive DDC tree__

<img src="https://raw.githubusercontent.com/subhh/HOS-TYPO3-discovery/master/screenshots/ss03.png"
width=620 />

__Wordcloud of subjects__

<img src="https://raw.githubusercontent.com/subhh/HOS-TYPO3-discovery/master/screenshots/ss04.png"
width=620 />

## Architecture
### LAMP

The architecture is based on [LAMP](https://en.wikipedia.org/wiki/LAMP_(software_bundle))
LAMP is an archetypal model of web service stacks, named as an acronym of the names of its original four open-source components: the Linux operating system, the Apache HTTP Server, the MySQL relational database management system (RDBMS), and the PHP programming language.

Unfortunately the developing system and the production system are based on different distributions of linux. Therefore we have different configurations and [package managing](https://www.digitalocean.com/community/tutorials/package-management-basics-apt-yum-dnf-pkg). Ubuntu uses [apt-get](https://www.digitalocean.com/community/tutorials/how-to-manage-packages-in-ubuntu-and-debian-with-apt-get-apt-cache), CENTOS uses [yum](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-centos-7).

The LAMP installation for [UBUNTU you can find here](Dokumentation/Ubuntu.md) and for [CentOS here](Dokumentation/CentOS.md).

Criteria of completeness:
* mySQL server is running with database `typo3`, admin user and typo3 user,
* PHP7.2 is installed with mysql, xml, json, gd, imagemagick, png, jpeg, gif
* up and running http servers

After this distribution depending basic installations you can continue with [installation of solr](Documentation/Solr.md) and the [installation TYPO3-app](Documentation/Discovery.md). 

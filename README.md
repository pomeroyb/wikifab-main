# Wikifab Installation

This project documents the installation of a wikifab style website

There are 2 methods to install Wikifab:

* METHOD #1 uses a full package to upload to your server using FTP. This method is described in detail on [Wikifab's original fork](https://github.com/Wikifab/wikifab-main). It has been removed from this repository in favor of installing the latest version using Method #2

* METHOD #2 uses Composer: it enables you to get the latest version of Wikifab, but it requires to have an ssh access to your server, with connectivity to download all the packages. Some web providers doesn't allow it.


## Installation Process Using Composer

### Requirement

You need a web server with PHP>5.4 with acces to execute php scripts

### 1. Download Mediawiki

*Wikifab uses Mediawiki version 1.31.3. Using another version may result in one or more of the extenstions failing to work.*

In bash : 

	wget https://releases.wikimedia.org/mediawiki/1.31/mediawiki-1.31.3.tar.gz
	tar -xzf mediawiki-1.31.3.tar.gz
	mv mediawiki-1.31.0 /var/www/yourwebsite

### 2. Install Your Wiki

Go to your website url, and follow installation instructions.

*Note : wikifab is only available in english and french for now. If you select another language, you will have a lot of missing translations.*

At the end of the installation, it should give you a file "LocalSettings.php" to put in your website directory.

At this point, your wiki is up, but it does not include the wikifab part.


### 3. Download wikifab-main

download this project, and copy content into your website folder

in bash :

	wget https://github.com/pomeroyb/wikifab-main/archive/master.zip
	unzip master.zip
	cp -R wikifab-main-master/* /var/www/yourwebsite/
	
### 4. Download and Run Composer

Download composer and execute composer.phar update into your website directory. This will get all extensions needed.
As Wikifab has not yet a fully stable version, you need to set "minimum-stability" to "dev" in composer.json

*Note: Do not run any of the following commands with sudo! Composer is designed to run in user-space*

in bash:

	cd /var/www/yourwebsite
	curl http://getcomposer.org/installer | php
	php composer.phar config minimum-stability dev
	php composer.phar update --no-dev

### 5. Move Flow Extension
The Flow extension installed by composer is not in the good directory, move it to 'extensions/' dir :
	
in bash:

	cd /var/www/yourwebsite
	mv vendor/mediawiki/flow extensions/Flow
	

### 6. Install Wikifab Extensions

In your "LocalSettings.php" file, add a line to include  file 'LocalSettings.wikifab.php'

	include('LocalSettings.wikifab.php');

To enable internationalization support, add the following line 

	include('LocalSettings.i18n.php');

and run the php update script 


in bash :

	cd /var/www/yourwebsite
	echo "include('LocalSettings.wikifab.php');" >> LocalSettings.php
	echo "include('LocalSettings.i18n.php');" >> LocalSettings.php
	php maintenance/update.php

### 7. Install Wikifab pages and formatting

You need to create all pages and forms to finish installation and have a wikifab like website.

You can do id with this script (only available for french for now) :

	php maintenance/initWikifab.php --setWikifabHomePage --int

Warning : this will change the home page of your wiki, if you do not want this, simply execute the command without param "--setWikifabHomePage"

### 8. Permissions

Finaly, make sure that server has write permissions on directories "images/" and "images/avatars/".

# Dependencies
### Inkscape is required for ImageAnnotator to work
    sudo apt install inkscape

### Visual Editor
Download Visual Editor version REL1_31
Go to [this page](https://www.mediawiki.org/wiki/Special:ExtensionDistributor?extdistname=VisualEditor&extdistversion=REL1_31) to find the correct link if the below code does not work
    
	wget https://extdist.wmflabs.org/dist/extensions/VisualEditor-REL1_31-c3c9140.tar.gz

Extract the contents into the extensions directory, e.g

    tar -xzf VisualEditor-REL1_31-c3c9140.tar.gz -C /var/www/mediawiki/extensions

### Install Parsoid 0.9 and NodeJS 10
*NOTE: Parsoid is very version sensitive. You must use the above versions if working with Mediawiki 1.31.3*

NodeJS

    sudo apt-get remove nodejs npm  
    curl -sL deb.nodesource.com/setup_10.x | sudo -E bash - 
	sudo apt-get install -y nodejs

Parsoid

    sudo apt install dirmngr
    sudo apt-key advanced --keyserver keys.gnupg.net --recv-keys AF380A3036A03444

*If the last command above isn't working (gpgkeys: key AF380A3036A03444 canot be retrieved), you can try another key server*

    sudo apt-key advanced --keyserver pgp.mit.edu --recv-keys AF380A3036A03444

Download and install Parsoid 0.9

    wget https://releases.wikimedia.org/parsoid/parsoid_0.9.0all_all.deb
	sudo dpkg -i parsoid_0.9.0all_all.deb
	sudo apt-get install -f

[Configure Parsoid using the instructions on this page](https://www.mediawiki.org/wiki/Parsoid/Setup#Configuration)
* If you started with a bitnami AWS stack, your entry point will probably be *'http://localhost/wiki/api.php'*



## Recommendations

### Recaptcha

To avoid bots to spam your pages, we strongly recommand you to activate captcha.

We recomend you recaptcha : to set it up : 
* go to https://www.google.com/recaptcha/ and after login, click "getrecatpha", and add an entry with your url. This will give you a key and secret key
* Edit LocalSettings.php, add the following lines, and replace with your key and secret key : 
	wfLoadExtensions( array( 'ConfirmEdit', 'ConfirmEdit/ReCaptchaNoCaptcha' ) );
	$wgCaptchaClass = 'ReCaptchaNoCaptcha';
	$wgReCaptchaSiteKey = 'YOUR-KEY';
	$wgReCaptchaSecretKey = 'YOUR-SECRET-KEY';

### Extension:3D

For wiki support for uploading and viewing 3D models, you need to install the 3D extension. To do so, add the following line to your LocalSettings file :

include('LocalSettings.3d.php');

with bash :

echo "include('LocalSettings.3d.php');" >> LocalSettings.php

#### Install 3d2png

3d2png is the thumbnail renderer for 3D files. It will render png thumbnails exactly like this extension will display the objects, using the same JS libraries running in Node.js instead of the browser.

To install, clone and install the 3d2png repository: 

	git clone https://gerrit.wikimedia.org/r/p/3d2png
	cd 3d2png
	npm install

On Linux, you'll also need to install a virtual framebuffer in order for 3d2png to be able to headlessly capture the 3D object.

	apt-get install xvfb

After having successfully installed 3d2png, we'll need to tell Extension:3D how to call this thumbnail generator service. Add this to your LocalSettings.php, and make sure to update the paths to match your configuration: 

$wg3dProcessor = [ '/usr/bin/xvfb-run', '-a', '-s', '-ac -screen 0 1280x1024x24' ,'/path-to-your-repository/3d2png.js' ];

## TroubleShooting

### you have a white page

This can means many differents errors.

To display errors messages, add the following lines on top of the LocalSettings.php file : 

	error_reporting( -1 );
	ini_set( 'display_errors', 1 );
	$wgShowExceptionDetails = true;

---
layout: post
title: Installing Xdebug for PHP7
category: php7, xdebug
---

Xdebug deepens debugging PHP apps and websites to a level you can’t receive from the manual process of using code level var_dump().

## Setup

To install Xdebug for PHP7 on Ubuntu you will need to do so manually. Ubuntu 15 and lower will not come with a package for PHP7 or its xDebug counterpart.

First, be sure you are using PHP 7 already by checking your version.

```
php -v
```

Should give you something like:

```
PHP 7.0.0-2+deb.sury.org~trusty+1 (cli) ( NTS )
Copyright (c) 1997-2015 The PHP Group
...
```

In my case I’m using the Larval Homestead PHP 7 branch with Vagrant and the VirtualBox provider. So, I have PHP 7 already installed and working with [Nginx](https://www.nginx.com/).

### Get your php.ini

Next output your php.ini information into a file or place you can get to the information from. I like to save mine to a file called php-info.txt.

```
sudo php -i > ~/php-info.txt
```

## Use the Xdebug Wizard

Send the text file information into the wizard at [Xdebug Wizard](http://xdebug.org/wizard.php). Then follow the instructions the wizard supplies.

### Example

The instructions should look something like:

    Download xdebug-2.4.0rc3.tgz (I like to use wget -O ~/downlaods/xdebug-2.4.0rc3.tgz http://xdebug.org/files/xdebug-2.4.0rc3.tgz on Ubuntu)
    Unpack the downloaded file with tar -xvzf xdebug-2.4.0rc3.tgz
    Run: cd xdebug-2.4.0rc3
    Run: phpize (See the Xdebug FAQ if you don’t have phpize.)

As part of its output it should be like:

Configuring for:
...
Zend Module Api No:      20151012
Zend Extension Api No:   320151012

If it does not, you are using the wrong phpize. Please follow this [FAQ entry](http://xdebug.org/docs/faq#custom-phpize) and skip the next step.

    Run: ./configure
    Run: make
    Run: cp modules/xdebug.so /usr/lib/php/20151012
    Edit /etc/php/7.0/cli/php.ini and add the line
    zend_extension = /usr/lib/php/20151012/xdebug.so

## Enable Remote Xdebug

To use[Xdebug remote debugging on a host computer you need to enable remote debugging on the guest server. In my case the guest is the Vagrant Homestead VM.

In the guest php.ini file add:

xdebug.remote_enable = 1
xdebug.remote_connect_back=1
xdebug.remote_port = 9000
xdebug.scream=0
xdebug.show_local_vars=1
xdebug.idekey=PHPSTORM

Since I use [PhpStorm for Debugging](https://www.youtube.com/watch?v=rqDDJfG6ip4) I set my Xdebug key to PHPSTORM. In the browser the Xdebug and PhpStorm will look for a cookie called XDEBUG_SESSION with the value PHPSTORM. To set this cookie I use the Chrome Browser extension [Xdebug helper](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc).

## Reboot Services

To load the new configurations reload your PHP and HTTP server services. In my case that is nginx and php7.0-fpm.

sudo service php7.0-fpm restart
sudo service nginx restart

## Configure PHP Storm

Now that Xdebug is install we are ready to configure a "PHP Remote Debug" for PhpStorm.

    Under PhpStorm > Preferences > Languages & Frameworks > PHP > Debug be sure Xdebug is set to port 9000
    Under PhpStorm > Preferences > Languages & Frameworks > PHP > Servers add a server using port 80 and set the Absolute path on the server.
    Under Run > Edit Configurations... add a "PHP Remote Debug"
    Set the server and the Ide key(session id) to PHPSTORM

## Using PhpStorm with Xdebug on a Remote Server

To start receiving requests from the remote server enable "Debug" with Xdebug helper in Google Chrome on the page you want to debug. Then, unlike local testing that uses "Start Listening for PHP Debug Connections" use Run > Debug instead.

Using the "Debug" command will open up the signal manually and open the direct connection to the remote server.

Set breakpoints and you are off.

### Out of Network Servers

If you are trying to remote debug a website with Xdebug from outside your local network be sure the 9000 port is not blocked on your network. This is typically done at the router level.

As always consult the [Xdebug remote documentation](http://xdebug.org/docs/remote).

Next, ask Google "what is my public ip" and it will tell you.

Then be sure your remote server is not blocking your IP address.

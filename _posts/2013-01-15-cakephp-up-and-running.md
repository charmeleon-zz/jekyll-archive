---
layout: post
title: "CakePHP Up and Running"
tagline: "And it only hurt a little"
description: ""
category: 
tags: [cakephp, php, ubuntu, debian]
---
{% include JB/setup %}

After two hours of aimlessly stumbling with CakePHP I've managed the task of
getting the framework up and running. Take that, 15 minute framework!

I've written before about the time that I broke my Ubuntu install, and I
attribute half of my headaches today on that fateful day \(but I could be wrong
\). For the other half, I will say that every tutorial I've come across
references the file httpd.conf, while Debian uses a different approach to
setting up a working Apache environment. Since I'm not a Linux admin this took
a while to figure out, so I'm  documenting what happened in hopes that if
another soul manages the same screw up, this post should halve the time for
figuring it out.

Setting up CakePHP on a Debian build
====================================

Let's take it form the top, more or less. I'm assuming that you already
downloaded [CakePHP](http://cakephp.org/) and unzipped the files, if necessary.
The next move would be to move them to a different place, I would suggest
`/var/www/`. Ideally, you should take the time to rename the folder to whatever
your app is called. Since I'm planning on working through the CakePHP tutorial,
I'm naming mine cakeblog:
> mv cakephp/ /var/www/cakeblog/

Now we want to let Apache know that we want to make this folder accessible
from the browser. Almost every tutorial out there says to change your
`httpd.conf` file, but Ubuntu __does not have an httpd.conf file__. Instead,
we setup under /etc/apache2/sites-available/. If you cannot find this path, try
the command `locate sites-available` which should point you in the right
direction. If you still cannot find this path, make sure that apache2 is
properly installed.

From sites-available, let's take the default file and make a copy of it:
> sudo cp /etc/apache2/sites-available/default /etc/apache2/sites-available/cakeblog

Now let's edit the latter file with the appropriate settings.
> sudo vim /etc/apache2/sites-available/cakeblog
First, change the DocumentRoot to read
> DocumentRoot /var/www/cakeblog/app/webroot
> ServerName cakeblog

The first line sets the root directory for our CakePHP application. The second
one identifies our VirtualHost. 

In this same file, on the section that reads `<Directory /var/www/>`, should
read like this \(of course, replace cakeblog with your application's name\):

> &lt;Directory /var/www/cakeblog/app/webroot/&gt;  
>    Options Indexes FollowSymLinks MultiViews  
>    AllowOverride All  
>    Order allow,deny  
>    allow from all  
> &lt;/Directory&gt;
which takes care of a few settings required by CakePHP. You're halfway there!

Next, let's edit your hosts file \(command `sudo vim /etc/hosts`\), and
redirect 127.0.0.1 to http://cakeblog via this line:
> 127.0.0.1 cakeblog

Then, again on the terminal, run this command to make a symlink to
 /etc/apache2/sites-enabled/:
> sudo a2ensite cakeblog

If you'd like to test your installation, point your web browser to
`http://localhost/cakeblog` \(or whatever name you gave it\). If it's up and
running - including CSS files - congratulations, you're done! Otherwise, let's
keep plowing through.

First, let's enable the mod rewrite module in Apache, which is required by
CakePHP.  
> sudo a2enmod rewrite
At this point I saw a blank page, and when I did "View Source", I saw PHP code
that was returned as regular text \(not preprocessed\). If this is happening
to you, try restarting the server. If it persists, try this magic sauce:
> apt-get install libapache2-mod-php5 php5-cli php5-common php5-cgi
This makes sure that __all__ the required PHP binaries are installed, and
anything you already have installed is ignored. Now, let's enable the PHP
module. From the terminal, let's find the Debian equivalent of httpd.conf:
> locate apache2.conf
Let's edit the file:
> sudo vim /etc/apache2/apache2.conf

This is a large file, so be careful to not touch too much. Insert the following
snippet towards the end of the file \(but before the `Include` statements\):
> \# Enable PHP 5  
> &lt;FilesMatch \.php$&gt;  
>   SetHandler application/x-httpd-php  
> &lt;/FilesMatch&gt;

Now let's enable PDO for the php.ini files found under apache2 and for php5-cli
> locate php5.ini
> sudo vim /etc/php5/apache2/php.ini
Look for the section `[Pdo]` and insert the following:
> extension=pdo.so
> extension=pdo_mysql.so

Do likewise for /etc/php5/cli/php.ini. Now we need to restart the apache server
for the changes to take place.
> sudo service apache2 restart
And that should be it. Congratulations! Your CakePHP install is up and running!

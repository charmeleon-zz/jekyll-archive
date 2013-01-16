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
getting the framework up and running. I've previously written on how I broke
my Ubuntu install awhile ago, and I strongly suspect this was one of many
sources for my figurative headaches today. The other was that every tutorial
I've come across references the file httpd.conf, while Debian uses a much
different approach to setting up a working Apache environment. So I'm 
documenting what happened in hopes that if another soul manages the same screw
up, this post should halve the time for figuring it out.

Setting up CakePHP on a (slightly broken) Debian build
====================================

Let's take it form the top, more or less. I'm assuming that you already
downloaded [CakePHP](http://cakephp.org/) and unzipped the files, if necessary.
The next move would be to move them to a different place, I would suggest
`/var/www/`. Ideally, you should take the time to rename the folder to whatever
your app is called. Since I'm planning on working through the CakePHP tutorial,
I'm naming it cakeblog:
> mv cakephp/ /var/www/cakeblog/

Now we want to let Apache know that we want to make this folder accessible
from the browser. Almost every tutorial out there says to change your
`httpd.conf` file, but Ubuntu __does not have an httpd.conf file__. Instead,
run this command:
> locate sites-available
and take note of the given path for the default site. Now let's make a copy of it.  
> sudo cp /etc/apache2/sites-available/default /etc/apache2/sites-available/cakeblog

Now let's edit the latter file with the appropriate settings.
> sudo vim /etc/apache2/sites-available/cakeblog
Let's change the DocumentRoot to read
> DocumentRoot /var/www/cakeblog/app/webroot/
Now, on the section that reads `<Directory /var/www/>`, should read like this
\(of course, replace cakeblog with your application's name\):
> &lt;Directory /var/www/cakeblog/app/webroot/&gt;  
>    Options Indexes FollowSymLinks MultiViews  
>    AllowOverride All  
>    Order allow,deny  
>    allow from all  
> &lt;/Directory&gt;
which takes care of a few settings required by CakePHP. You're almost there!

If you'd like to test your installation, point your web browser to
`http://localhost/cakeblog` \(or whatever name you gave it\). If it's up and
running - including CSS files - congratulations, you're done! Otherwise, let's
keep plowing through.

First, let's enable the mod rewrite module in Apache, which is required by
CakePHP.  
> sudo a2enmod rewrite
If your build is really broken then this isn't the end. At this point I saw a
blank page, and when I did "View Source", I saw PHP code that was returned as
regular text \(not preprocessed\). For that reason, try this magic sauce:
> apt-get install libapache2-mod-php5 php5-cli php5-common php5-cgi
This makes sure that __all__ the required thingees are installed, and anything
you already have installed is ignored. Now, let's enable the PHP module. From
the terminal, let's find the Debian equivalent of httpd.conf:
> locate apache2.conf
Let's edit the file:
> vim /etc/apache2/apache2.conf

This is a large file, so be careful to not touch too much. Insert the following
snippet towards the end of the file \(but before the `Include` statements\):
> \# Enable PHP 5  
> &lt;FilesMatch \.php$&gt;  
>   SetHandler application/x-httpd-php  
> &lt;/FilesMatch&gt;

Now we need to restart the apache server for the changes to take place.
> sudo service apache2 restart
And that should be it. Congratulations! Your CakePHP install is up and running!

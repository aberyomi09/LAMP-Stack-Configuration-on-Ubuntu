**STEP 1 — INSTALLING APACHE AND UPDATING THE FIREWALL**

**What exactly is Apache?**

Apache HTTP Server is the most widely used web server software. Developed and
maintained by Apache Software Foundation, Apache is an open source software
available for free. It runs on 67% of all webservers in the world. It is fast,
reliable, and secure. It can be highly customized to meet the needs of many
different environments by using extensions and modules. Most WordPress hosting
providers use Apache as their web server software. However, websites and other
applications can run on other web server software as well. Such as Nginx,
Microsoft’s IIS, etc.

The Apache web server is among the most popular web servers in the world. It’s
well documented, has an active community of users, and has been in wide use for
much of the history of the web, which makes it a great default choice for
hosting a website.

**Install Apache using Ubuntu’s package manager ‘apt’**:

**update a list of packages in package manager**

*\#sudo apt update*

**run apache2 package installation**

*\#sudo apt install apache2*

**To verify that apache2 is running as a Service in our OS, use following
command**

*\#sudo systemctl status apache2*

Before we can receive any traffic by our Web Server, we need to open TCP port 80
which is the default port that web browsers use to access web pages on the
Internet

As we know, we have TCP port 22 open by default on our EC2 machine to access it
via SSH, so we need to **add a rule to EC2 configuration to open inbound
connection through port 80**:

![Inbound rules](https://user-images.githubusercontent.com/66855448/152492344-dbcaad5c-5990-4dbd-8148-2da513ee2cb0.PNG)

Our server is running and we can access it locally and from the Internet (Source
0.0.0.0/0 means ‘from any IP address’).

First, let us try to check how we can access it locally in our Ubuntu shell,
run:

*\#curl http://localhost:80*

OR

*\#curl http://127.0.0.1:80*

These 2 commands above actually do pretty much the same – they use ‘curl’
command to request our Apache HTTP Server on port 80 (actually you can even try
to not specify any port – it will work anyway). The difference is that: in the
first case we try to access our server via DNS name and in the second one – by
IP address (in this case IP address 127.0.0.1 corresponds to DNS name
‘localhost’ and the process of converting a DNS name to IP address is called
"resolution").

Now it is time for us to test how our Apache HTTP server can respond to requests
from the Internet.

Open a web browser of your choice and try to access following url

*http://\<Public-IP-Address\>:80*

Another way to retrieve your Public IP address, other than to check it in AWS
Web console, is to use following command:

*\#curl -s http://169.254.169.254/latest/meta-data/public-ipv4*

The URL in browser shall also work if you do not specify port number since all
web browsers use port 80 by default.

If you see following page, then your web server is now correctly installed and
accessible through your firewall.

![apache_ubuntu_default](https://user-images.githubusercontent.com/66855448/152492394-50c816b7-ef34-4745-b399-de23b3c606ad.png)

**STEP 2 — INSTALLING MYSQL**

Now that you have a web server up and running, you need to install a Database
Management System (DBMS) to be able to store and manage data for your site in a
relational database. MySQL is a popular relational database management system
used within PHP environments, so we will use it in our project.

Again, use ‘apt’ to acquire and install this software:

*\#sudo apt install mysql-server*

When prompted, confirm installation by typing Y, and then ENTER.

When the installation is finished, it’s recommended that you run a security
script that comes pre-installed with MySQL. This script will remove some
insecure default settings and lock down access to your database system. Start
the interactive script by running:

*\#sudo mysql_secure_installation*

![Mysql secure installation](https://user-images.githubusercontent.com/66855448/152492441-d7b77768-b51b-4fc8-8313-e8724314c148.PNG)

This will ask if you want to configure the VALIDATE PASSWORD PLUGIN.

Note: Enabling this feature is something of a judgment call. If enabled,
passwords which don’t match the specified criteria will be rejected by MySQL
with an error. It is safe to leave validation disabled, but you should always
use strong, unique passwords for database credentials.

Answer Y for yes, or anything else to continue without enabling.

If you answer “yes”, you’ll be asked to select a level of password validation.
Keep in mind that if you enter 2 for the strongest level, you will receive
errors when attempting to set any password which does not contain numbers, upper
and lowercase letters, and special characters, or which is based on common
dictionary words.

Regardless of whether you chose to set up the VALIDATE PASSWORD PLUGIN, your
server will next ask you to select and confirm a password for the MySQL root
user. This is not to be confused with the system root. The database root user is
an administrative user with full privileges over the database system. Even
though the default authentication method for the MySQL root user dispenses the
use of a password, even when one is set, you should define a strong password
here as an additional safety measure. We’ll talk about this in a moment.

If you enabled password validation, you’ll be shown the password strength for
the root password you just entered and your server will ask if you want to
continue with that password. If you are happy with your current password, enter
Y for “yes” at the prompt:

For the rest of the questions, press Y and hit the ENTER key at each prompt.
This will remove some anonymous users and the test database, disable remote root
logins, and load these new rules so that MySQL immediately respects the changes
you have made.

When you’re finished, test if you’re able to log in to the MySQL console by
typing:

*\#sudo mysql*

![sudo mysql](https://user-images.githubusercontent.com/66855448/152492491-a11af7a0-acad-4fc1-a8d6-1a51b3a86240.PNG)

This will connect to the MySQL server as the administrative database user root,
which is inferred by the use of sudo when running this command. You should see
output like this:

To exit the MySQL console, type:

*\#mysql\> exit*

Notice that you didn’t need to provide a password to connect as the root user,
even though you have defined one when running the mysql_secure_installation
script. That is because the default authentication method for the administrative
MySQL user is unix_socket instead of password. Even though this might look like
a security concern at first, it makes the database server more secure because
the only users allowed to log in as the root MySQL user are the system users
with sudo privileges connecting from the console or through an application
running with the same privileges. In practical terms, that means you won’t be
able to use the administrative database root user to connect from your PHP
application. Setting a password for the root MySQL account works as a safeguard,
in case the default authentication method is changed from unix_socket to
password.

For increased security, it’s best to have dedicated user accounts with less
expansive privileges set up for every database, especially if you plan on having
multiple databases hosted on your server.

Note: At the time of this writing, the native MySQL PHP library mysqlnd doesn’t
support caching_sha2_authentication, the default authentication method for MySQL
8\. For that reason, when creating database users for PHP applications on MySQL
8, you’ll need to make sure they’re configured to use mysql_native_password
instead.

Your MySQL server is now installed and secured. Next, we will install PHP, the
final component in the LAMP stack.

**STEP 3 — INSTALLING PHP**

You have Apache installed to serve your content and MySQL installed to store and
manage your data. PHP is the component of our setup that will process code to
display dynamic content to the end user. In addition to the php package, you’ll
need php-mysql, a PHP module that allows PHP to communicate with MySQL-based
databases. You’ll also need libapache2-mod-php to enable Apache to handle PHP
files. Core PHP packages will automatically be installed as dependencies.

**To install these 3 packages at once, run:**

*\#sudo apt install php libapache2-mod-php php-mysql*

Once the installation is finished, you can run the following command to confirm
your PHP version:

*\#php –v*

![php -v](https://user-images.githubusercontent.com/66855448/152492563-30acf367-4691-4e34-bc44-68e234e79920.PNG)

**At this point, your LAMP stack is completely installed and fully
operational.**

**Linux (Ubuntu)**

**Apache HTTP Server**

**MySQL**

**PHP**

To test your setup with a PHP script, it’s best to set up a proper Apache
Virtual Host to hold your website’s files and folders. Virtual host allows you
to have multiple websites located on a single machine and users of the websites
will not even notice it.

![Virtual host](https://user-images.githubusercontent.com/66855448/152492827-7c4c6f0d-4540-4f3b-9efe-856cd815b97d.png)

We will configure our first Virtual Host in the next step.

**STEP 4 — CREATING A VIRTUAL HOST FOR YOUR WEBSITE USING APACHE**

In this project, you will set up a domain called projectlamp, but you can
replace this with any domain of your choice.

Apache on Ubuntu 20.04 has one server block enabled by default that is
configured to serve documents from the */var/www/html* directory.

We will leave this configuration as is and will add our own directory next next
to the default one.

**Create the directory for projectlamp using ‘mkdir’ command as follows:**

*\#sudo mkdir /var/www/projectlamp*

**Next, assign ownership of the directory with your current system user:**

*\#sudo chown -R \$USER:\$USER /var/www/projectlamp*

Then, create and open a new configuration file in Apache’s sites-available
directory using your preferred command-line editor. Here, we’ll be using vi or
vim (They are the same by the way):

*\#sudo vi /etc/apache2/sites-available/projectlamp.conf*

This will create a new blank file. Paste in the following bare-bones
configuration by hitting on i on the keyboard to enter the insert mode, and
paste the text:

*\<VirtualHost \*:80\>*

*ServerName projectlamp*

*ServerAlias www.projectlamp*

*ServerAdmin webmaster@localhost*

*DocumentRoot /var/www/projectlamp*

*ErrorLog \${APACHE_LOG_DIR}/error.log*

*CustomLog \${APACHE_LOG_DIR}/access.log combined*

*\</VirtualHost\>*

**You can use the ls command to show the new file in the sites-available
directory**

*\#sudo ls /etc/apache2/sites-available*

You will see something like this;

**000-default.conf default-ssl.conf projectlamp.conf**

With this VirtualHost configuration, we’re telling Apache to serve projectlamp
using /var/www/projectlamp as its web root directory. If you would like to test
Apache without a domain name, you can remove or comment out the options
ServerName and ServerAlias by adding a \# character in the beginning of each
option’s lines. Adding the \# character there will tell the program to skip
processing the instructions on those lines.

You can now use a2ensite command to enable the new virtual host:

*\#sudo a2ensite projectlamp*

You might want to disable the default website that comes installed with Apache.
This is required if you’re not using a custom domain name, because in this case
Apache’s default configuration would overwrite your virtual host. **To disable
Apache’s default website use a2dissite command , type:**

*\#sudo a2dissite 000-default*

**To make sure your configuration file doesn’t contain syntax errors, run:**

*\#sudo apache2ctl configtest*

**Finally, reload Apache so these changes take effect:**

*\#sudo systemctl reload apache2*

Your new website is now active, but the web root /var/www/projectlamp is still
empty.

**Create an index.html file in that location so that we can test that the
virtual host works as expected:**

*\#sudo echo 'Hello LAMP from hostname' \$(curl -s
http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP'
\$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) \>
/var/www/projectlamp/index.html*

**Now go to your browser and try to open your website URL using IP address:**

*http://\<Public-IP-Address\>:80*

If you see the text from ‘echo’ command you wrote to index.html file, then it
means your Apache virtual host is working as expected.

In the output you will see your server’s public hostname (DNS name) and public
IP address. You can also access your website in your browser by public DNS name,
not only by IP – try it out, the result must be the same (port is optional)

*http://\<Public-DNS-Name\>:80*

You can leave this file in place as a temporary landing page for your
application until you set up an index.php file to replace it. Once you do that,
remember to remove or rename the index.html file from your document root, as it
would take precedence over an index.php file by default.

**STEP 5 — ENABLE PHP ON THE WEBSITE**

With the default DirectoryIndex settings on Apache, a file named index.html will
always take precedence over an index.php file. This is useful for setting up
maintenance pages in PHP applications, by creating a temporary index.html file
containing an informative message to visitors. Because this page will take
precedence over the index.php page, it will then become the landing page for the
application. Once maintenance is over, the index.html is renamed or removed from
the document root, bringing back the regular application page.

In case you want to change this behavior, **you’ll need to edit the
/etc/apache2/mods-enabled/dir.conf** file and change the order in which the
index.php file is listed within the DirectoryIndex directive:

*\#sudo vim /etc/apache2/mods-enabled/dir.conf*

*\<IfModule mod_dir.c\>*

*\#Change this:*

*\#DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm*

*\#To this:*

*DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm*

*\</IfModule\>*

**After saving and closing the file, you will need to reload Apache so the
changes take effect:**

*\#sudo systemctl reload apache2*

Finally, we will create a PHP script to test that PHP is correctly installed and
configured on your server.

Now that you have a custom location to host your website’s files and folders,
we’ll create a PHP test script to confirm that Apache is able to handle and
process requests for PHP files.

**Create a new file named index.php inside your custom web root folder:**

*\#vim /var/www/projectlamp/index.php*

This will open a blank file. Add the following text, which is valid PHP code,
inside the file:

*\<?php*

*phpinfo();*

When you are finished, save and close the file, refresh the page and you will
see a page similar to this:

![phpinfo](https://user-images.githubusercontent.com/66855448/152492766-7f2d0b8f-c2e1-431c-b33e-a99403e36fde.png)

This page provides information about your server from the perspective of PHP. It
is useful for debugging and to ensure that your settings are being applied
correctly.

If you can see this page in your browser, then your PHP installation is working
as expected.

After checking the relevant information about your PHP server through that page,
it’s best to remove the file you created as it contains sensitive information
about your PHP environment -and your Ubuntu server. You can use rm to do so:

*\#sudo rm /var/www/projectlamp/index.php*

You can always recreate this page if you need to access the information again
later.

**Conclusion**

Now that we have a LAMP stack installed, we have many choices for what to do
next. We’ve installed a platform that will allow us to install most kinds of
websites and web software on our server.

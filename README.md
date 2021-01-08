# Linux Server Configuration

A step by step on how to configure an Amazon Lightsail Ubuntu Linux instance
 to deploy the web application [Item Catalog](https://github.com/dtmarangoni/item_catalog).
 
Apache web server was chosen to serve the application.

Technologies involved: Linux Server configuration like packages installation 
and updates, firewalls, SSH, user management, timezone setting, PostgreSQL, 
Apache Web Server and Let's Encrypt with certbot for HTTPS enabling.


## Specific Configuration for Udacity project
(**As the course ended the server has been shutting down**).

For Udacity project evaluation the following configuration was used:
- Amazon Lightsail Ubuntu 18.04 server image;
- Linux user: grader;
- Server public ip address: 54.166.176.230;
- SSH port: 2200;
- Enabled ports in firewall: 2200, 80, 123 and 443;
- The HTTPS port (443) was enabled in order to allow implementation of an HTTPS website;
- The website URL address:
	- https://54.166.176.230.xip.io

## Installation Steps

### 1 - Get a Linux Server Instance
Get an [Amazon Lightsail](https://aws.amazon.com/pt/lightsail/) Ubuntu 18.04
 server instance ready for use.

Take note of:
- your instance public ip address;
- default user name for ssh connection;
- download the private key for ssh key-pair connection.

### 2 - Connecting and Configuring the Server
First connection from terminal using SSH:

`ssh user_name@public_ip_address -i your_LightsailDefaultKey.pem`

#### 2.1 - Updating and Installing Packages in the System
Modules necessary in order to Item Catalog project work:
- `sudo apt-get -y update`;
- `sudo apt-get -y upgrade`;
- `sudo apt-get -y install python3-pip`;
- `sudo pip3 install --upgrade pip`;
- `sudo apt-get -y install git`;
- `sudo apt-get -y install apache2`;
- `sudo apt-get -y install libapache2-mod-wsgi-py3`;
- `sudo apt-get -y install postgresql`;
- `sudo apt-get -y install libpq-dev`;
- `sudo pip3 install psycopg2`;
- `sudo pip3 install sqlalchemy`;
- `sudo pip3 install passlib`;
- `sudo pip3 install itsdangerous`;
- `sudo pip3 install flask`;
- `sudo pip3 install flask_httpauth`;
- `sudo pip3 install oauth2client`;
- `sudo apt-get -y install software-properties-common`;
- `sudo add-apt-repository universe -y`;
- `sudo add-apt-repository ppa:certbot/certbot -y`;
- `sudo apt-get update`;
- `sudo apt-get -y install certbot python-certbot-apache`;

#### 2.2 - Configuring the Firewall and SSH port
##### 2.2.1 - Configuring Lightsail UFW Firewall
- `sudo ufw status`;
- `sudo ufw default deny incoming`;
- `sudo ufw default allow outgoing`;
- `sudo ufw allow 2200/tcp`;
- `sudo ufw allow www`;
- `sudo ufw allow https`;
- `sudo ufw allow ntp`;

##### 2.2.2 - Changing the SSH Default Port from 22 to 2200
Replace the `# Port 22` by `Port 2200` in:
- `sudo nano /etc/ssh/sshd_config`.

Enable user and password login method while we don't have key-pair 
authentication configured.
- Where you find **_# To disable tunneled clear text passwords, change to no 
here!_**, set: 
    `PasswordAuthentication yes`.

Restart ssh service:
- `sudo service ssh restart`.

##### 2.2.3 - Now Enable the Firewall
- `sudo ufw enable`;
- `sudo ufw status`.


#### 2.3 - Creating a Linux User
Instead of using the default Lightsail user and private key let's define ours.

##### 2.3.1 - Adding a New User
- `sudo adduser user_name`.

##### 2.3.2 - Giving the User sudo Permission
- `sudo touch /etc/sudoers.d/user_name`;
- Insert the text `user_name ALL=(ALL) NOPASSWD:ALL` in:
    - `sudo nano /etc/sudoers.d/user_name`.

##### 2.3.3 - Generating SSH key-pair
Exit to local system and do:
- `ssh-keygen`. Enter the desired path, key name and passphrase.

The program will generate two files: a public and a private key.

##### 2.3.4 - Connect to the Server with the New User
- `ssh user_name@public_ip_address -p 2200`.

Place the public key contents in the server to allow key based 
authentication. Inside of your */home* directory do:
- `mkdir .ssh`;
- `touch .ssh/authorized_keys`;
- Contents of the public key to be placed here:
    - `nano .ssh/authorized_keys`.

Now change the authorized_keys and it's folder permissions:
- `chmod 700 .ssh`;
- `chmod 644 .ssh/authorized_keys`.

For security reasons, turn off user and password login method again:
- `sudo nano /etc/ssh/sshd_config`.
- Where you find **_# To disable tunneled clear text passwords, change to no 
here!_**, set:
    - `PasswordAuthentication no`.

Also disable root login:
- Where you find **_#PermitRootLogin prohibit-password_**, set:
    - `PermitRootLogin no`.

Restart the ssh service:
- `sudo service ssh restart`.

You can now exit and test the new method of connection:
- `ssh user_name@public_ip_address -p 2200 -i 
/path_to_your_private_key/private_key`.

#### 2.4 - Changing System Timezone to UTC
First check if date is already synced with UTC:
- `date`;
- `timedatectl`;
- If necessary activate timedatectl service:
    - `sudo timedatectl set-ntp on`.
- Check it now: `timedatectl`.

#### 2.5 - Packages Upgrade and System Restart
Before continuing to project installation, if there's still 
packages pending
 of upgrade, do:
- `sudo apt-get update`;
- `sudo apt-get dist-upgrade`.

If a system restart is needed, do:
- `sudo reboot`.

### 3 - Clone the Item Catalog Project
In your home folder or another place of your preference do:
- `git clone https://github.com/dtmarangoni/item_catalog.git`.

Add JSON files containing client secrets information. This project implement connections to Facebook and Google.

Access the links to create a web application and oauth credentials:
- [Facebook Developers](https://developers.facebook.com/). Include the https://your_public_ip_address.xip.io as site URL;
- [Google Developers Console](https://console.developers.google.com). Include https://your_public_ip_address.xip.io and https://your_public_ip_address as Javascript authorized origins and https://your_public_ip_address.xip.io as authorized URIs for redirection.
- After finished, download the JSON file from Google and place it in 
*static/json* inside of project folder. Rename it to *google_client_secrets
.json*.
- The Facebook JSON file must be manually created. A template example is 
already inside the folder - */static/json/facebook_client_secrets.json*.

### 4 - Create the Database and Database User
#### 4.1 - Defining the Database and User in PostgresSQL
- `sudo su postgres -c 'createdb item_catalog'`;
- `sudo su postgres -c "psql -c \"CREATE USER catalog WITH PASSWORD 
'icproject';\""`.

#### 4.2 - Create and Populate the Database Tables
Inside the project folder perform the steps below:
- Creating the database tables:
    - `python3 database.py`;
- Populate the database with some example categories for initial data:
    - `python3 populate_db.py`.

### 5 - Configuring Apache

#### 5.1 - Create and Configure the HTTPS Certificate
Create and configure the HTTPS certificate with Let's Encrypt and certbot.

Certbot has an Apache plugin, which is supported on many platforms, and 
automates certificate installation.
- You will need to answer some questions like your e-mail, your project 
domain address and if you want to restrict the access to both HTTP and HTTPS or
 HTTPS only.
- After done you will have the path to your Lets Encrypt folder, certificate
 and key file.
- Command to run the plugin:
    - `sudo certbot --apache`.
- If you want to make the changes to your Apache configuration by hand, you 
can use the certonly subcommand:
    - `sudo certbot --apache certonly`.

#### 5.2 - Configuring Apache to Handle Requests using the WSGI Module
- `cd /etc/apache2/sites-available`;
- `sudo nano catalog-ssl.conf`;
- Add the following content inside of *catalog-ssl.conf*. Replace the 
necessary info with your configuration:
    - your_public_ip_address;
    - path_to_your_project;
    - path_to_your_cert_file;
    - path_to_your_cert_key;
    - path_to_your_lets_encrypt;
    - user_name
```
<IfModule mod_ssl.c>
	<VirtualHost *:443>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		CustomLog ${APACHE_LOG_DIR}/access.log combined

		RewriteEngine on

		ServerName your_public_ip_address.xip.io
		DocumentRoot /path_to_your_project
		SSLCertificateFile /path_to_your_cert_file/your_public_ip_address.xip
		.io/fullchain.pem
		SSLCertificateKeyFile /path_to_your_cert_key/your_public_ip_address.xip.io/privkey.pem
		Include /path_to_your_lets_encrypt/options-ssl-apache.conf

		WSGIDaemonProcess catalog user=user_name group=user_name threads=5
		WSGIScriptAlias / /path_to_your_project/application.wsgi

		<Directory /path_to_your_project>
				WSGIProcessGroup catalog
				WSGIApplicationGroup %{GLOBAL}
				Options Indexes FollowSymLinks
				AllowOverride None
				Require all granted
		</Directory>
		
	</VirtualHost>
	
</IfModule>
```

#### 5.3 - Enable the Newly Created VirtualHost
- `sudo a2ensite catalog-ssl.conf`;
- Restarting Apache:
    - `sudo systemctl reload apache2`.


## Accessing the Website
Now you can access the website from browser:

https://your_public_ip_address.xip.io

In case of any problem or error, they can be checked in this log:
- `sudo cat /var/log/apache2/error.log`.


## References
### 1 - Changing the Default SSH Port
https://br.godaddy.com/help/alterar-a-porta-ssh-para-o-seu-servidor-linux-7306

### 2 - Changing the Timezone to UTC
https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt

https://www.digitalocean.com/community/tutorials/how-to-set-up-time-synchronization-on-ubuntu-18-04

### 3 - PostgreSQL
#### 3.1 - Connections and Authentication
https://www.postgresql.org/docs/current/runtime-config-connection.html

#### 3.2 - Tutorial to Roles and Permissions
https://www.digitalocean.com/community/tutorials/how-to-use-roles-and-manage-grant-permissions-in-postgresql-on-a-vps--2

### 4 - Flask and Apache Deploy
http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/

http://www.devfuria.com.br/python/flask-apache/

http://httpd.apache.org/docs/2.4/

### 5 - Supporting HTTPS with Let's Encrypt and certbot:
https://letsencrypt.org/getting-started/

https://certbot.eff.org/lets-encrypt/ubuntubionic-apache

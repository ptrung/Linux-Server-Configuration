# Linux-Server-Configuration

This projects descripes the baseline installation of a Linux server for hosting the web application: [Item-Catalog](https://github.com/ptrung/Item-Catalog)

* IP-Adress: 18.184.11.54
* Port: 2200
* URL: http://ec2-18-184-11-54.eu-central-1.compute.amazonaws.com/

## Walkthrough
### Get and access the server
1. Start a new Ubuntu Linux server instance on [Amazon Lightsail](https://lightsail.aws.amazon.com/)
2. Download the private key from the SSH-Key section in the Account section on Amazon Lightsail. 
3. Move you private key to `~/.ssh/`
4. Change the permission of the private key with: `chmod 400 <public-key-file>`
5. Access the server with the command: `ssh -i ~/.ssh/<public-key> ubuntu@<ip>`

### Update packages
1. Update and upgrade all installed packages: 
`sudo apt-get update`
`sudo apt-get upgrade`

### Configurate SSH Port and Firwall
1. To change the SSH Port: `sudo nano /etc/ssh/sshd_config` and change Port 22 to 2200 and change PermitRootLogin to no
2. Reload SSH with `sudo service ssh restart`
3. Go to the *network* configuration of your Lightsail server and change the ports of the firewall to TCP/80, TCP/2200 and UDP/123.
4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123): 
`sudo ufw allow 2200/tcp`
`sudo ufw allow 80/tcp`
`sudo ufw allow 123/udp`
`sudo ufw enable`

### Create new user 'grader' with sudo permisson
1. `sudo adduser grader`
2. `sudo nano /etc/sudoers.d/grader` 
3. Add content `grader ALL=(ALL) NOPASSWD:ALL` to this file and save it

### Create SSH Key
1. Run `ssh-keygen` on your local machine to create a keypair 
2. Copy the content of the public-key *<key>.pub*
3. Run `sudo nano .ssh/authorized_keys`
4. Add the copied content from 2 into this file and save it
5. Change the permissions: 
`chmod 700 .ssh`
`chmod 644 .ssh/authorized_keys`
6. Login is now possible with: `ssh -i ~/.ssh/<private-key> -p 2200 grader@18.184.11.54`

### Change timezone
1. `sudo timedatectl set-timezone Etc/UTC`

### Install and configurate Apache
1. Install Apache: `sudo apt-get install apache2`
2. Install mod_wsgi: `sudo apt-get install libapache2-mod-wsgi python-dev`
3. Enable mod_wsgi: `sudo a2enmod wsgi`
4. Start Apache: `sudo service apache2 start`

### Install PostgreSQL
1. Install PostgreSQL: ´sudo apt-get install postgresql´
2. Check if no remote connections are allowed `sudo vim /etc/postgresql/9.5/main/pg_hba.conf`
3. Login as user "postgres": ´sudo su - postgres´
4. Get into postgreSQL shell: `psql`
5. Create a new database named catalog: `CREATE DATABASE catalog;`
6. Create a new user named catalog: `CREATE USER catalog;`
7. Set password to catalog: `ALTER ROLE catalog WITH PASSWORD 'password';`
8. Give user permission to database: `GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`
9. Quit postgreSQL shell: `\q`
10. Exit from user "postgres": `exit`

### Setup project
1. Install git: ´sudo apt-get install git´
2. Change directory: ´cd /var/www`
3. Clone project: `git clone https://github.com/ptrung/Item-Catalog.git`
4. Rename folder: `mv Item-Catalog catalog`
4. Go to new directory: `cd catalog`
5. Raname *application.py*: ´sudo mv application.py __init__.py`
6. Edit *__init__.py*, *database_setup.py* and *dummy_data.py* and change `engine = create_engine('sqlite:///catalog.db')` to ´engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
7. Change client_secrets.json path to `/var/www/catalog/catalog/client_secrets.json` (in *__init__.py*)
8. Install pip: `sudo apt-get install python-pip`
9. Install dependencies: `sudo pip install Flask oauth2client sqlalchemy psycopg2 sqlalchemy_utils requests`
10. Create database schema `sudo python database_setup.py`
11. Fill database `sudo python dummy_data.py`

## Configure and enable a new virtual host
1. Run this: ´sudo nano /etc/apache2/sites-available/catalog.conf´
2. Paste this code: 
  ```
<VirtualHost *:80>
	ServerName 18.184.11.54
	ServerAdmin admin@18.184.11.54
	WSGIScriptAlias / /var/www/catalog/catalog.wsgi
	<Directory /var/www/catalog/catalog/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/catalog/catalog/static
	<Directory /var/www/catalog/catalog/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
  ```
3. Enable the virtual host `sudo a2ensite catalog`
4. Create .wsgi File: `sudo nano /var/www/catalog/catalog.wsgi`
5. Paste this code: 
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = '<key>'
  ```
6. Apache neustarten: `sudo service apache2 restart`

## Reference
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

## Acknowledge

This project was built as part of the Udacity course "Fullstack Web Developer Nanodegree". 

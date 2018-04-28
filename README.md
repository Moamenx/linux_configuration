# Udacity Full Stack Web Developer Nanodegree - Linux Server Configuration Project

## 1. Details specific to the server I set up
The IP address is 18.184.106.193

The SSH port used is `2200`.

The URL to the hosted webpage is: http://18.184.106.193/ 


## 2. Software required
- Apache2
- mod_wsgi
- PostgreSQL
- git
- pip
- virtualenv
- httplib2
- Python Requests
- oauth2client
- SQLAlchemy
- Flask
- libpq-dev
- Psycopg2


## 3. Configuration steps
### Create an instance with Amazon Lightsail
1. Sign in to [Amazon Lightsail](https://amazonlightsail.com) using an Amazon Web Services account

1. Follow the 'Create an instance' link

1. Choose the 'OS Only' and 'Ubuntu 16.04 LTS' options

1. Choose a payment plan

1. Give the instance a unique name and click 'Create'

1. Wait for the instance to start up


## 4. Server Configuration and Super user creation

Before doing any work we need to update all packages by doing the following:

- `sudo apt update`
- `sudo apt upgrade`

Then we can start doing the required work: 

1. Create a New User grader and give grader sudo 'sudo adduser grader'

2. Create a new file in the sudoers directory with 'sudo nano /etc/sudoers.d/grader'

3. Add the following text 'grader ALL=(ALL) NOPASSWD ALL'

4. Change the SSH port from 22 to 2200 'sudo nano /etc/ssh/sshd_config' 

5. Restart ssh service 'sudo service ssh restart '

6. Create an SSH key pair for grader using the 'ssh-keygen' (On the local machine) and call it anything (I called it key)

7. Create a diriectory for the new user with .ssh 'sudo mkdir /home/grader/.ssh'

8. Change permission and give it to the new user (grader) 'sudo chown grader:grader /home/grader/.ssh '

9. Change folder permission 'sudo chmod 700 /home/grader/.ssh'    

10. Copy the authorized_keys to /grader/.ssh 'sudo cp /home/ubuntu/.ssh/authorized_keys /home/grader/.ssh/'

11. Change permission of the authorized_keys file 'sudo chmod 644 /home/grader/.ssh/authorized_keys'

12. Update the Amazon Lightsail firewall on the browser by clicking on the 'Manage' option, then the 'Networking' tab, and then changing the firewall configuration to match the internal firewall settings above (only ports `80`(TCP), `123`(UDP), and `2200`(TCP) should be allowed; make sure to deny the default port `22`)

13. You can now disconnect from the browser-based connection by typing 'exit'

14. now you can use ssh to login with the new user you created 'ssh -i [privateKeyFilename] -p 2200 grader@18.184.106.193'

15. Baaam, you are logged in using the key authorization method with your new grader user.


------

## Deploy Catalog Application

We will use a virtual machine, apache2, mod_wsgi and postgre to host our application. Before we any thing, ssh to the Amazon Terminal through your Terminal with grader ('ssh -i [privateKeyFilename] -p 2200 grader@18.184.106.193')

1. Install required packages

- `sudo apt install apache2`
- `sudo apt install libapache2-mod-wsgi python-dev`
- `sudo apt install git`

2. Enable mod_wsgi by `$ sudo a2enmod wsgi` 
3. Start the web server by `sudo service apache2 start` or `sudo service apache2 restart`. 
4. You should input the public IP  address and you should see the default apache html page. By going to your browser and 'http://18.184.106.193/'


5. Set up the folder structure (although clumsy, but works)
- `$ cd /var/www`
- `$ sudo mkdir catalog`
- `$ sudo chown -R grader:grader catalog`
- `$ cd catalog`

6. Now we clone the project from Github: `$ git clone [your link] catalog` Copy your link from here is the easiet way:
![github git link](https://github.com/callforsky/udacity-linux-configuration/blob/master/pic/pic11.png)

7. Create a .wsgi file: `$sudo nano catalog.wsgi` and add the following into this file
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'supersecretkey'
```

6. Rename the `RestaurantMenuApplication.py` to `__init__.py`

7. Now we need to install and start the virtual machine
- `sudo pip install virtualenv`
- `sudo virtualenv venv`
- `source venv/bin/activate`
- `sudo chmod -R 777 venv`

You should see a `(venv)` appears before your username in the command line:

8. Now we need to install the Flask and other packages needed for this application
- `sudo apt-get install python-pip`
- `sudo pip install Flask`
- `sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlaclemy_utils requests render_template, redirect, psslib [anything else you have built within this application`

9. Use the `nano __init__.py` command to change the `client_secrets.json` line to `/var/www/catalog/catalog/client_secrets.json`  and change the host to your Amazon Lightsail public IP address and port to 80
![change host and port](

10. Now we need to configure and enable the virtual host
- `$ sudo nano /etc/apache2/sites-available/catalog.conf`
- Paste the following code and save
```
<VirtualHost *:80>
    ServerName [YOUR PUBLIC IP ADDRESS]
    ServerAlias [YOUR AMAZON LIGHTSAIL HOST NAME]
    ServerAdmin admin@35.167.27.204
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
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

11. Now we need to set up the database
- `sudo apt install libpq-dev python-dev`
- `sudo apt install postgresql postgresql-contrib`
- `sudo su - postgres -i`

You should see the username changed again in command line, and type `$ psql` to get into postgres command line

12. Now we create a user to create and set up the database. I name my database `catalog` with user `catalog`
- `CREATE USER catalog WITH PASSWORD [your password] (in my case grader);`
- `ALTER USER catalog CREATEDB;`
- `CREATE DATABASE catalog WITH OWNER catalog;`
- Connect to database `$ \c catalog`
- `REVOKE ALL ON SCHEMA public FROM public;`
- `GRANT ALL ON SCHEMA public TO catalog;`
- Quit the postgrel command line: `$ \c` and then `$ exit`

13. use `sudo nano` command to change all engine to `engine = create_engine('postgresql://catalog:[your password]@localhost/catalog`

13. Restart Apache server `sudo service apache2 restart` and enter your public IP address or host name into the browser.

## Reference
I deeply thank to those alumni who posted their step-by-step on their Github:
- https://github.com/rrjoson/udacity-linux-server-configuration/blob/master/README.md
- https://github.com/stueken/FSND-P5_Linux-Server-Configuration
- https://github.com/iliketomatoes/linux_server_configuration



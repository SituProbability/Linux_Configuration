# Udacity Full Stack Web Developer Nanodegree - Linux Server Configuration Project

This project prepare a Linux user to host a web applications.

URL: http://52.27.202.6.xip.io

IP Address: 52.27.202.6

## Amazon Lightsail Server Set Up

1. Visit Amazon Lightsail and create a new AWS account

2. After log in, click 'Create Instance'

3. Select Platform (linux) and  blueprint (ubuntu)

4. Scroll down to name the instance and click 'Create'

5. After the instant is runing, click on Networking to create static IP

## Server Configuration

1. Connect to the instance using an SSH client application like PuTTY
- Download default private key and save the lightsailDefaultKey.pem key.
- Save private key using PuTTYgen. 
- Open Putty and type the static IP address and the default port will be 22.
- Load private key log in to the instance using the default user name - ubuntu.

2. Create another user 'grader'
- Create a new key pair in Lightsail Account page: choose Create key pair, name the pair, choose Generate key pair, and then save a copy to your local machine.
- Get a copy of public key using PuTTYgen. Choose Save private key with the name grader. Confirm that don't want to save it with a passphrase.
- Connect to the instance with user ubuntu.
- Type `$ sudo adduser grader`.
- Create a new file in the sudoers directory: `$ sudo nano /etc/sudoers.d/grader`. And give grader the super permisssion `grader ALL=(ALL:ALL) ALL`.
- Switch into user grader `$ sudo su - grader`
- Create a .ssh directory: `$ mkdir .ssh`.
- Create a file to store the public key: `$ touch .ssh/authorized_keys`.
- Edit the authorized_keys file `$ nano .ssh/authorized_keys` by adding the public key to it.
- Change the permission: `$ sudo chmod 700 /home/grader/.ssh` and `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`.
- Restart the ssh service: `$ sudo service ssh restart`.
- Now you can log in as grader with the grader private key.

3. Enforce the key-based authentication: `$ sud nano /etc/ssh/sshd_config`. Find the *PasswordAuthentication* line and change text to `no`. After this, restart ssh again: `$ sudo service ssh restart`.

4. Disable ssh login for *root* user to prevent attacks: `$ sudo nano /etc/ssh/sshd_config`. Find the *PermitRootLogin* line and edit to `no`. Restart ssh `$ sudo service ssh restart`.

5. On the Lightsail Manage page, choose Networking. Add these ruls to the Firewall: "Custom UDP 123", "Custom TCP 2222".

6. Disconnect the server and then log back through port 2200.

7. Configure Uncomplicated Firewall:
- `$ sudo ufw allow 2200/tcp`
- `$ sudo ufw allow 80/tcp`
- `$ sudo ufw allow 123/udp`
- `$ sudo ufw enable`

8. Run the following commands to update all packages and install finger package:
- `$ sudo apt-get update`
- `$ sudo apt-get upgrade`
- `$ sudo apt-get install finger`

## Deploy Catalog Application

1. Install required packages
- `$ sudo apt-get install apache2`
- `$ sudo apt-get install libapache2-mod-wsgi python-dev`

2. Enable mod_wsgi `$ sudo a2enmod wsgi` and start the web server by `$ sudo service apache2 start` or `$ sudo service apache2 restart`.

3. Enter your public IP address in your browser now and the apache2 default page should be loaded.

4. Create item_catalog folder to keep the app.
- `$ cd /var/www`
- `$ sudo mkdir item_catalog`
- `$ cd item_catalog`

5. Clone the project from Github (so folder path to app will become `var/www/item_catalog/item_catalog`)

6. Create a .wsgi file in `/var/www/item_catalog/`: `$sudo nano item_catalog.wsgi` and add the following into this file
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/item_catalog/")

from item_catalog import app as application
application.secret_key = 'super_secret_key'
```

7. In /var/www/item-catalog/item_catalog Rename the `views.py` to `__init__.py` as follows `mv views.py __init__.py`

8. Install virtual environment
- `$ sudo apt-get install python-pip`
- `$ sudo pip install virtualenv`
- `$ sudo virtualenv venv`
- `$ source venv/bin/activate`

9. Install the Flask and other packages needed for this application
- `$ sudo pip install Flask`
- `$ sudo pip install sqlalchemy sqlaclemy requests psycopg2`

10. Configure and enable the virtual host
- `$ sudo nano /etc/apache2/sites-available/catalog.conf`
- Paste the following code and save
```
<VirtualHost *:80>
    ServerName 52.27.202.6.xip.io
    ServerAdmin admin@52.27.202.6
    WSGIScriptAlias / /var/www/item_catalog/item_catalog.wsgi
    <Directory /var/www/item_catalog/item_catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/item_catalog/item_catalog/static
    <Directory /var/www/item_catalog/item_catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

12. Set up the database
- `$ sudo apt-get install libpq-dev python-dev`
- `$ sudo apt-get install postgresql postgresql-contrib`
- `$ sudo -su postgres`
You should see the username changed in command line, and type `$ psql` to get into postgres command line.

13. Create a user to create and set up the database.
- `$ CREATE USER catalog WITH PASSWORD [your password];`
- `$ ALTER USER catalog CREATEDB;`
- `$ CREATE DATABASE catalog WITH OWNER catalog;`
- Connect to database `$ \c catalog`
- `$ REVOKE ALL ON SCHEMA public FROM public;`
- `$ GRANT ALL ON SCHEMA public TO catalog;`
- Quit the postgrel command line: `$ \c` and then `$ exit`

14. Use `sudo nano` command to change all engine to `engine = create_engine('postgresql://catalog:[your password]@localhost/catalog`

15. Initiate the database: `python models.py `

16. Use the `nano __init__.py` command to change the `client_secrets.json` line to `/var/www/item_catalog/item_catalog/client_secrets.json` as follows 
    `CLIENT_ID = json.loads(open('/var/www/item_catalog/item_catalog/client_secrets.json', 'r').read())['web']['client_id']`
    Ensure to look through `__ini__.py` for every instance of this change and replace as stated.
    Also replace
    ```if __name__ == '__main__':
    app.secret_key = 'super_secret_key'
    app.debug = True
    app.static_folder = 'static'
    app.run(0.0.0.0, port=8000)```
    with
    ```if __name__ == '__main__':
    app.secret_key = 'super_secret_key'
    app.debug = True
    app.static_folder = 'static'
    app.run()```

17. Update Google Oauth 2.0 credential
- Add URL `http://52.27.202.6.xip.io` to the JavaScript Origins and redirect URIs.
- Also use `nano` command to update client_secrets.json file. 

18. Enabled the app by executing `sudo a2ensite [name of app]`

19. Restart Apache server `$ sudo service apache2 restart` and enter the Url into the browser. The application is online!

## Appendix:
I. Some apache2 testing tools to check appropriateness of workflow
`apache2ctl -t`
`apache2ctl -S`

II. Check Main ErrorLog
`nano /var/log/apache2/error.log`

## Reference
- https://www.youtube.com/watch?v=Ta7HAvu-GPs&t=1437s
- https://www.youtube.com/watch?v=-LwI4HMR_Eg
- https://github.com/judekuti/Linux-Configuration
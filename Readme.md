# Linux Server Configuration

This project includes baseline installation of a Linux server and prepare it to host Item Catalog Web Application,  securing server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

## 1.Get Your Server

1. Signup or login to amazon web services 
2. Create an instance
3. Choose an instance image : ubuntu
4. Choose instance plan
5. Give your instance a host name
6. That's it.. now you have your server up and running

## 2.Secure your server

1. **Update all currently installed packages**
   1. `$ sudo apt-get update`
   2. `$ sudo apt-get upgrade`
2. **Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it**.
   1. `$ sudo nano /etc/ssh/sshd_config ` and change port from 22 to 2200`
3. **Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).**
   1. `$ sudo ufw default deny incoming`
   2. `$ sudo ufw default allow outgoing`
   3. `$ sudo ufw allow ssh`
   4. `$ sudo ufw allow 2200/tcp`
   5. `$ sudo ufw allow www`
   6. `$ sudo ufw allow ntp`
   7. `$ sudo ufw allow 123/udp`
   8. restart ssh service `sudo service ssh restart`

## 3. Give `grader` access.

1. ### create new user account named `grader`

   1. ### `$ sudo adduser grader`

2. **Give `grader` the permission to `sudo`.**

   1. `$ sudo nano /etc/sudoers` inside this file under "user privileges section" after "root ALL=(ALL:ALL) ALL" write "grader ALL=(ALL:ALL) NOPASSWD:ALL" . 

3. **Create an SSH key pair for `grader` using the `ssh-keygen` tool.**

   1. Inside local system generate key `ssh-keygen` 

   2. give file a name (project_key) and store it in `~/.ssh/project_key`

   3. Now inside `~/.ssh/` there will be one file with `*.pub` extension

   4. This is public key which you have to store in server so

   5. Now switch to grader `$ sudo su - grader`

   6. Create `.ssh` folder and `.ssh/authorized_keys` file `$ mkdir .ssh` and `$ touch .ssh/authorized_keys` 

   7. type `sudo nano .ssh/authorized_keys` and then paste the content of public key generated locally

   8. give `.ssh` and `.ssh/authorized_keys ` permissions `$ chmod 700 .ssh ` and `$ chmod 644 .ssh/authorized_keys`

   9. `$ exit`

   10. **Now Login With `grader` and `key`**

     1. `ssh grader@13.127.192.1 -p 2200 -i [Private Key File Name]`


   11. Now You are logged in with `grader` with `sudo` access using private key.

12. ## **Prepare to Deploy Project**

    1. **Configure the local time zone to UTC.**

       1. `$ sudo dpkg-reconfigure tzdata`

       2. Now choose from `None of the above` and after choosing None You will get UTC option. (default)

          ​

    2. **install and configure Apache to serve a Python mod_wsgi application**

       1. `$ sudo apt-get install apache2`
       2. `$ sudo apt-get install libapache2-mod-wsgi-py3` For python 3

    3. **Install and configure PostgreSQL**

       1. `$ sudo apt-get install postgresql`
       2. `$ sudo apt-get install psycopg2`
       3. `$ sudo apt-get install psycopg2-binary`
       4. Do not allow remote connections
          1. `$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf` verify no remote connections are allowed (default: not allowed)
       5. **Create a new database user named `catalog` that has limited permissions to your catalog application database.**
          1. `$ sudo su - postgres`
          2. type `psql`
          3. type `# CREATE DATABASE item_catalog;`
          4. type `# CREATE USER catalog;`
          5. type `# ALTER ROLE catalog WITH PASSWORD 'catalog';`
          6. type `GRANT ALL PRIVILEGES ON DATABASE item_catalog TO catalog;`
          7. `# \q`
          8. `$ exit`
          9. now project database is ready.
       6. **Install git** `sudo apt-get install git`

13. ##  Deploy the Item Catalog project

    1. **Clone and setup your `Item Catalog` project from the `Github repository` you created earlier in this `Nanodegree program`**

       1. Move to "/var/www" directory. `$ cd /var/www`

       2. Create Folder  `flask_app`. `$ mkdir flask_app`

       3. clone `item_catalog repo`. `$ git clone https://github.com/shivam19597/item_catalog.git`

       4. All the changes required for deployment is already done in repo.


   2. **Install Dependencies**

      1. `$ sudo apt-get install python3`
      2. `$ sudo apt-get install python3-pip`
      3. `$ sudo pip install flask sqlalchemy httplib2 requests oauth2client`

   3. **Set Up and Load Database**

      1. `$ sudo python3 database_setup.py`
      2. `$ sudo python3 load_sample_data.py`

   4. **Create .WSGI file**

      1. change directory to `/var/www/flask_app/item_catalog`

      2. create `item_catalog.wsgi` file. `sudo touch item_catalog.wsgi`

      3. Edit `item_catalog.wsgi`file. `sudo nano item_catalog.wsqi`

      4. Copy and Paste Following Line to `.wsgi` file

         1. ```
            #!/usr/bin/python
            import sys
            import logging
            logging.basicConfig(stream=sys.stderr)
            sys.path.insert(0,"/var/www/flask_app/item_catalog/")

            from server import app as application
            application.secret_key = 'secret key'
            ```

   5. **Configure a New Virtual Host**

      1. change directory to `/etc/apache2/sites-available`. `$ cd /etc/apache2/sites-available`

      2. Inside This Folder create `.conf` file for virtual host configuration

         1. `sudo nano flask_app.conf`

            1. copy and paste following code

            2. ```
               <VirtualHost *:80>
               	ServerName http://ec2-13-127-192-1.ap-south-1.compute.amazonaws.com
               	ServerAlias 13.127.192.1
               	ServerAdmin talk2shivamsharma19597@gmail.com
               	WSGIScriptAlias / /var/www/flask_app/item_catalog/flask_app.wsgi
               	<Directory /var/www/flask_app/item_catalog/>
               		Order allow,deny
               		Allow from all
               	</Directory>
               	Alias /static /var/www/flask_app/item_catalog/static
               	<Directory /var/www/flask_app/item_catalog/static/>
               		Order allow,deny
               		Allow from all
               	</Directory>
               	ErrorLog ${APACHE_LOG_DIR}/error.log
               	LogLevel warn
               	CustomLog ${APACHE_LOG_DIR}/access.log combined
               </VirtualHost>
               ```

         ​

   6. **Restart SSH**

      1. `sudo service ssh restart`

   7. **Restart Apache2** 

      1. `sudo service apache2 restart`

8. ## Development Environment information

   1. #### **domain name**

      1. #### [**ec2-13-127-192-1.ap-south-1.compute.amazonaws.com**]( http://ec2-13-127-192-1.ap-south-1.compute.amazonaws.com)

   2. ##### Public I.P address

      1. ##### 13.127.192.1

   3. ###  `grader` private key

      1. inside project folder `linux_server_configuration`  file name : `project_key`

   4. **`grader` passphrase**

      ​	"passphrase"

   5. **SSH PORT:**

      1. 2200

9. **Google `oauth` authentication  **

   1. From [This Link](http://www.hcidata.info/host2ip.cgi) Get your domain name for static ip.
   2. Go to google developer console and in credential section change javaScript origin and redirect uri to generated domain name
   3. Apache2 is already configured for this domain name.

## References:

<https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps>

<https://modwsgi.readthedocs.io/en/develop/>

<http://www.hcidata.info/host2ip.cgi>

<https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts>

<http://docs.sqlalchemy.org/en/latest/core/engines.html>

<http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/>

<https://aws.amazon.com/documentation/lightsail/>

<http://fosshelp.blogspot.com/2014/03/how-to-deploy-flask-application-with.html>


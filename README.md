 # linux-server-conf-instruction

 Detailed instruction on how to set up and configure an Ubuntu Linux Web server with using
 Amazon Lightsail service, Flask, SQLAlchemny, PostgreSQL, Apache. This is the last project 
 in Udacity Full Stack Nanodegree program. 


 ## General
 URLs:
   1. http://ec2-35-154-151-158.ap-south-1.compute.amazonaws.com/
   2. http://35.154.151.158/
  
 Public IP: 35.154.151.158

 SSH Port: 2200
  
 All steps below are oriented for macOS users, but I believe they are should be the same for
 Windows and especially Linux users.

 ## Initialisation
 Go to [Amazon Lightsail], then
   1. Create new account or sign in.
   2. Start a new Ubuntu Linux server instance and follow the steps during creating of the instance:
        - chose *OS only* and *Ubuntu 16.04*
        - choose a payment plan
        - write an unique name of this instance
   3. Start the instance (if it didn't start automatically)
   
 #### Connection:
   There are a few options to work with your instance:
   1. Through an Amazon Lightsail web terminal (I found this not stable and not convenient at all):
        1) Open your Amazon Lightsail homepage
        2) Click "Resources" tab
        3) Find your instance there
        4) On the right top corner of it find a terminal/command line icon and click it
        5) Congrats you are inside and ready to go
   2. Through the terminal on your local machine
        There is an issue with some macOS, default key for ssh doesn't work correctly, so follow next steps to avoid that:  
        1) Open Amazon Lightsail "Account page"
        2) Click on "Download default key"
        3) Open downloaded the file by any text editor 
        4) Copy text from there
        5) Create a file with format *.rsa* in directory *~/.ssh/*, for example *~/.ssh/my\_awesome\_key.rsa* 
        6) Paste copied text
        7) Now you need to change permission for this file, type in your terminal:
            ``` 
            $ chmod 600 ~/.ssh/lightrail_key.rsa
            ```
        8) Now you are ready to log in, type next lines:
           ```
           $ ssh -i ~/.ssh/my_awesome_key.rsa ubuntu@my_public_ip
           ```
           see the section "General" to check how my_public_ip should look, in my case it is *35.154.151.158*. 
           This key you can find on Amazon Lightsail homepage under tab "Resources" near the name of your instance.
            

 #### Update all currently installed packages.
 Before start your work you need to update all packages to prevent any errors in the future.

 Type in your terminal next commands:
  ```
  $ sudo apt-get update
  $ sudo apt-get upgrade
 ```

 #### Change timezone to UTC:
 1) Type: 
     ```
     $ sudo dpkg-reconfigure tzdata
     ```
 2) Select 'None of the above' on the first page, and on the second page select 'UTC'
 
 
 #### Configure the firewall:
 1) Open */etc/ssh/sshd\_config* find the line 5 and change there 22 to 2200, then type:
      ```
      $ sudo service ssh restart
      ```
 2) Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
      ```
      $ sudo ufw default deny incoming
      $ sudo ufw default allow outgoing
      $ sudo ufw allow ssh
      $ sudo ufw allow 2200/tcp
      $ sudo ufw allow www
      $ sudo ufw allow 123/udp
      $ sudo ufw deny 22
      $ sudo ufw enable
      ```
      **Note:** Use ```sudo ufw status``` command to check UFW status and allowed/denied ports
      
 3) Go to Lightsail instance homepage and open "Networking" tab, update settings like so: 
      1) ports 80(TCP), 123(UDP) and 2200(TCP) - allowed
      2) port 22 - denied
 4) Now to log in via terminal on your mac use next command: 
      ```
      $ ssh -i ~/.ssh/my_awesome_key.rsa -p 2200 ubuntu@my_public_ip
      ```
 

 **Important:** For all next steps you have to be connected to your instance.  

 ## Create user
 #### Create user and give him "sudo" permissions
 Username will be **grader**, and password can be any:
 
  1) Create user:
     ```
     $ sudo adduser grader
     ```
  2) Create a new file in the */etc/sudoers.d* directory: 
     ```
     $ sudo nano /etc/sudoers.d/grader
     ```
  3) Add this line to the file to give **grader** sudo permissions:
   
     **"grader ALL=(ALL:ALL) ALL"**
  4) Save it.
 
 More details: [Configuring Linux Web Servers | Udacity]
 
 
 #### Create an SSH key pair for grader using the ssh-keygen tool
 1) Generate a key pair on your local machine with: 
    ```
    $ ssh-keygen -f ~/.ssh/grader-access_key
    ```
 2) Copy content of grader-access_key.pub, which was created but previous command
 3) Log in to the VM 
 4) Type next command to create authorized_key file for grader:
    ```
    $ touch /home/grader/.ssh/authorized_keys
    ```
 5) Open this file and paste the content from step 2 there
 6) Change permission:
    ```
    $ chmod 700 .ssh
    $ chmod 644 .ssh/authorized_keys
    ```
 7) To enforce key based authentication:
     1) Log in as grader
     2) Open sshd_config file:
        ```
        $ sudo nano /etc/ssh/sshd_config
        ```
     3) Find line "# Change to no to disable tunnelled clear text passwords", if on the next line
        you see "PasswordAuthentication yes" change "yes" to "no"
     4) Restart ssh service: 
        ```
        $ sudo service ssh restart
        ```
 8) You should be able to log in now as grader via:
    ```
    $ ssh -i ~/.ssh/grader-access_key -p 2200 grader@my_public_ip
    ```
     
 ## Install and configure tools
 #### Git
 Type in your terminal next command:
 ```
 $ sudo apt-get install git
 ```
 
 #### Apache
 Type in your terminal next command:
 ```
 $ sudo apt-get install apache2
 ```
 
 Visit your public IP in your favourite browser, you should see there "Ubuntu page"
 
 #### mod_wsgi
 Type in your terminal next command:
 ```
 $ sudo apt-get install libapache2-mod-wsgi python-dev
 ```
 
 With this package your Apache will be able to serve Flask application.
 
 To enable wsgi run next command:
 ```
 $ sudo a2enmod wsgi
 ```
 
 #### PostgreSQL
 Type in your terminal next command:
 ```
 $ sudo apt-get install postgresql
 ``` 
 Now you need to create a new PostgreSQL user, you can name it whatever you want, 
 in my case I named it *catalogitems* as a name of my project for simplicity. So let's
 get started.
  
 1) First of all you need to change your current user to postgres one, for that use:
  ```
    $ sudo su - postgres
  ```
 2) Type next command to connect to psql
  ```
    $ psql
  ```
 3) You are inside now and here you should use SQL language, to create user *catalogitems* type:
  ```
    CREATE ROLE catalogitems WITH LOGIN;
  ```
 4) To set the password for this user, type:
  ```
    \password catalog
  ```
 5) To give this user the ability to create databases, type:
  ```
    ALTER ROLE catalog CREATEDB;
  ```
 6) To return to the ubuntu user, type:
  ```
    \q
    exit
  ```

 
 ## Project preparation
 All next info related to my own project [flask-item-catalog], 
 so there is no need to use the same names of directories and files. Let's get started.
  
 #### Clone project
 1) From the root navigate to the directory /var/www:
 ```
 $ cd /var/www/
 ```
 2) Create a folder called  *flaskItemsCatalog* and go inside:
 ```
 $ mkdir flaskItemsCatalog
 $ cd flaskItemsCatalog
 ```
 3) Clone the project and create a name for the project dir:
 ```
 $ sudo git clone https://github.com/egor-sorokin/flask-items-catalog flaskItemsCatalog
 ```
 
 #### Update Google OAuth2
 Currently in the project Google OAuth2 was configured for **localhost:5000** and 
 it's OK for a local environment but now we have a real IP.
 1) Let's create a new project in the [Google developer console]: 
    To create new client ID use the instruction in this course [Udacity OAuth2] or these steps: but instead of **localhost:5000**
    1) Log in to [Google developer console]
    2) Open up credentials tab
    3) Select "Create Credentials"
    4) Choose "OAuth Client ID"
    5) Select "Web Application"
    6) Enter name of your project
    7) Add to authorized JavaScript origins:
        1) http://my_public_ip
        2) http://ec2-my_public_ip.ap-south-1.compute.amazonaws.com 
    8) Add as authorized redirect URIs:
        1) http://ec2-my_public_ip.ap-south-1.compute.amazonaws.com/login
        2) http://ec2-my_public_ip.ap-south-1.compute.amazonaws.com/oauth2callback
        3) http://ec2-my_public_ip.ap-south-1.compute.amazonaws.com/gconnect 
 
 2) Save these credentials on your local machine 
 3) Open and copy in a buffer all data there
 4) In the terminal (you should be inside of the instance) navigate to the project dir:
 ```
 $ cd /var/www/flaskItemsCatalog/flaskItemsCatalog
 ```
 5) Find **client\_secrets.json** file, open it and replace all data there by new credentials from buffer
 6) Find and open the file **views.py** and replace on lines 27 and 65 *client\_secrets.json* by 
 */var/www/flaskItemsCatalog/flaskItemsCatalog/client\_secrets.json*
   
 #### Update database name
 As now we use PostgreSQL instead of SQLite we need to change some lines in several files
 1) Navigate to *flaskItemsCatalog* dir and open **views.py** again
 2) Find line with **engine = create\_engine('sqlite:///catalog.db')** and replace it by 
    **engine = create\_engine('postgresql://catalogitems:passwowrd\_of\_your\_database@localhost/catalogitems')**
 3) Do the same for files **models.py** and **fake\_data.py**
 
 #### Other updates in the project's files
 1) Let's make the owner of the full project **ubuntu** user, we should do that [recursively] so type next command:
  ```
  $ cd /var/www/
  $ sudo chown -R ubuntu:ubuntu flaskItemsCatalog/
  ```
 2) Now change the name of **views.py** file to **\_\_init\_\_.py**
 ```
 $ cd /var/www/flaskItemsCatalog/flaskItemsCatalog
 $ mv view.py __init__.py
 ```
 3) Open this file and scroll to the end of it
 4) Find the line with **app.run(host='0.0.0.0', port=5000)** and remove **host='0.0.0.0', port=5000** there
 
 You are almost done!
 
 #### Install virtual env and project dependencies
 The last item in the whole section is a virtual environment and dependencies.
 1) Install pip and virtual env globally:
 ```
 $ sudo apt-get install python-pip
 $ sudo apt-get install python-virtualenv
 ```
 2) Create virtual environment in the directory */var/www/flaskItemsCatalog/flaskItemsCatalog*:
 ```
 $ virtualenv my_awesome_env_name
 ```
 3) Activate it:
 ```
 $ . my_awesome_env_name/bin/activate
 ```
 4) Now you are ready to install all dependencies:
 ```
 $ pip install sqlalchemy
 $ pip install flask
 $ pip install httplib2
 $ pip install requests
 $ pip install --upgrade oauth2client
 $ sudo apt-get install libpq-dev
 $ pip install psycopg2
 ```
 5) To deactivate the virtual environment type:
 ```
 $ deactivate
 ``` 
 
 
 [recursively]: <http://nersp.nerdc.ufl.edu/~dicke3/nerspcs/chown.html>
 [Udacity OAuth2]: <https://www.udacity.com/course/authentication-authorization-oauth--ud330>
 [Google developer console]: <https://console.developers.google.com/projectselector/apis/credentials>
 [flask-item-catalog ]: <https://github.com/egor-sorokin/flask-items-catalog>
 [Amazon Lightsail]: <https://amazonlightsail.com/
 [Configuring Linux Web Servers | Udacity]: <https://www.youtube.com/watch?v=axn-ni_NFoo&list=PLAwxTw4SYaPmH5HlJY4ABvoJ4jyGztDD7>
 

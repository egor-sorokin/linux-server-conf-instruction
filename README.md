 # linux-server-conf-instruction

 Detailed instruction on how to set up and configure an Ubuntu Linux Web server with using
 Amazon Lightsail service, Flask, SQLAlchemny, PostgreSQL, Apache. This is the last project 
 in Udacity Full Stack Nanodegree program. 


 ### General
 URLs:
   1. http://ec2-35-154-151-158.ap-south-1.compute.amazonaws.com/
   2. http://35.154.151.158/
  
 Public IP: 35.154.151.158

 SSH Port: 2200
  
 All steps below and oriented for macOS users, but I believe they are should be the same for
 Windows and especially Linux users.

 ### Initialisation
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
        3) Open downloaded file by any **text editor** 
        4) Copy text from there
        5) Create a file with format *.rsa* in directory *~/.ssh/*, for example *~/.ssh/my_awesome_key.rsa* 
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
 Before start your work you need to **update all packages to prevent any errors in the future**.

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
 1) Open */etc/ssh/sshd_config* find line 5 and change there 22 to 2200, then type:
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

 ### Create user
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
    chmod 700 .ssh
    chmod 644 .ssh/authorized_keys
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
     ssh -i ~/.ssh/grader-access_key -p 2200 grader@my_public_ip
     ```
 
 
 
 [Amazon Lightsail]: <https://amazonlightsail.com/
 [Configuring Linux Web Servers | Udacity]: <https://www.youtube.com/watch?v=axn-ni_NFoo&list=PLAwxTw4SYaPmH5HlJY4ABvoJ4jyGztDD7>
 
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
 
 
 
 [Amazon Lightsail]: <https://amazonlightsail.com/
 
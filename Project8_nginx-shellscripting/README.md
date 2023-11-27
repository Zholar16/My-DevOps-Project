# AUTOMATING LOAD-BALANCER CONFIGURATION WITH SHELL SCRIPTING

As a DevOps engineer, automation is the core of our responsibility. Automation helps us speed up deployment of services and reduces the chances of error in everyday activities in the CICD process.

# Automating webserver deployment

In the implementation, i deployed two backend servers with a load balancer distributing traffic across the webservers. This was accomplished using appropriate commands right on the terminal.

# Deploying and Configuring the webservers

## Step 1

* I provisioned two EC2 instances running ubuntu 22.

![Alt text](Images/5.png)

## Step 2

* I opened port 8000 to allow traffic from anywhere using the security group.

![Alt text](Images/6.png)

## Step 3

* I copnnected to the webserver via the terminal using SSH client

![Alt text](Images/7.png)

![Alt text](Images/8.png)

## Step 4 

* I opened a file, and pasted the script below using the command `sudo nano install.sh`


```#!/bin/bash

####################################################################################################################
##### This automates the installation and configuring of apache webserver to listen on port 8000
##### Usage: Call the script and pass in the Public_IP of your EC2 instance as the first argument as shown below:
######## ./install_configure_apache.sh 127.0.0.1
####################################################################################################################

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure

PUBLIC_IP=$1

[ -z "${PUBLIC_IP}" ] && echo "Please pass the public IP of your EC2 instance as an argument to the script" && exit 1

sudo apt update -y &&  sudo apt install apache2 -y

sudo systemctl status apache2

if [[ $? -eq 0 ]]; then
    sudo chmod 777 /etc/apache2/ports.conf
    echo "Listen 8000" >> /etc/apache2/ports.conf
    sudo chmod 777 -R /etc/apache2/

    sudo sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8000>/' /etc/apache2/sites-available/000-default.conf

fi
sudo chmod 777 -R /var/www/
echo "<!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my EC2 instance</h1>
            <p>Public IP: "${PUBLIC_IP}"</p>
        </body>
        </html>" > /var/www/html/index.html

sudo systemctl restart apache2

```
       

* I closed the file using `Ctl+ X + Y + Enter`


![Alt text](Images/9.png)


![Alt text](Images/10.png)


## Step 5

* I changed permissions on the file to make it executable using the command `sudo chmod +x install.sh`

## Step 6

* I run the shell script using the command below.

    `./install.sh public_IP` 

    `./install.sh 34.224.89.113:8000` 

    `./install.sh 107.21.76.35:8000`

* I inserted the `public IP address` of both webservers

![Alt text](Images/11.png)

![Alt text](Images/12.png)



# Deployment of Nginx as a load balancer using a shell script

Having successfully deployed and configured the two webservers. I provisioned another EC2 instance running ubuntu 22. I opened port 80 to anywhere using the security group and connected the loadbalancer via the terminal.

![Alt text](Images/13.png)

![Alt text](Images/16.png)

## Step 1

* I opened a file `nginx.sh` using the command sudo nano nginx.sh

![Alt text](Images/14.png)

## Step 2

* I copied and pasted the script below:

```
#!/bin/bash

######################################################################################################################
##### This automates the configuration of Nginx to act as a load balancer
##### Usage: The script is called with 3 command line arguments. The public IP of the EC2 instance where Nginx is installed
##### the webserver urls for which the load balancer distributes traffic. An example of how to call the script is shown below:
##### ./configure_nginx_loadbalancer.sh PUBLIC_IP Webserver-1 Webserver-2
#####  ./configure_nginx_loadbalancer.sh 127.0.0.1 192.2.4.6:8000  192.32.5.8:8000
############################################################################################################# 

PUBLIC_IP=$1
firstWebserver=$2
secondWebserver=$3

[ -z "${PUBLIC_IP}" ] && echo "Please pass the Public IP of your EC2 instance as the argument to the script" && exit 1

[ -z "${firstWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the second argument to the script" && exit 1

[ -z "${secondWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the third argument to the script" && exit 1

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure


sudo apt update -y && sudo apt install nginx -y
sudo systemctl status nginx

if [[ $? -eq 0 ]]; then
    sudo touch /etc/nginx/conf.d/loadbalancer.conf

    sudo chmod 777 /etc/nginx/conf.d/loadbalancer.conf
    sudo chmod 777 -R /etc/nginx/

    
    echo " upstream backend_servers {

            # your are to replace the public IP and Port to that of your webservers
            server  "${firstWebserver}"; # public IP and port for webserser 1
            server "${secondWebserver}"; # public IP and port for webserver 2

            }

           server {
            listen 80;
            server_name "${PUBLIC_IP}";

            location / {
                proxy_pass http://backend_servers;   
            }
    } " > /etc/nginx/conf.d/loadbalancer.conf
fi

sudo nginx -t

sudo systemctl restart nginx

```


## Step 3

* I closed the file using `Ctl + X + Y + Enter`

## Step 4

* I changed the file permission using the command `sudo chmod +x nginx.sh`

## Step 5 

* I run the script with the command below:

    `./nginx.sh 3.89.247.111 34.224.89.113:8000 107.21.76.35:8000`

![Alt text](Images/15.png)


# Verifying the Set up

* I got the following screenshots after verification

Webserver 1

![Alt text](Images/1.png)

Webserver 2

![Alt text](Images/2.png)

Load balancer

![Alt text](Images/3.png)

![Alt text](Images/4.png)
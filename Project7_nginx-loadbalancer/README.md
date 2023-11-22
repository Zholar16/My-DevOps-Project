# IMPLEMENTATION OF LOAD BALANCING WITH NGINX

## Introduction to Load Balancing and Nginx

__What is a Load Balancer__

A load balancer distributes workloads across multiple compute resources, such as virtual servers. Using a load balancer increases the availability and fault tolerance of your applications.

You can add and remove compute resources from your load balancer as your needs change, without disrupting the overall flow of requests to your applications.

The load balancer stands in front of the webserver and all traffic gets into it first, it then distribute the traffic evenly across the set of webservers. This ensures no webserver gets overworked,consequently improving performance.

Nginx is a versatile software it acts like a webserver,reverse proxy, and a load balancer etc.All that is needed is to configure it properly to server to your use case.

## Setting Up a BAsic Load Balancer

__Step 1: Provisioning EC2 Instance__

* I opened AWS management console and and clicked on Launch instance

* I provided unique names to ther servers namely: 
    * webserver 1
    * webserver 2
    * loadbalancer

* I clicked on Quick Start and chose Ubuntu under Applications and OS images

* I used an existing key pair `webstack_key`

* Finally, i launched the instance

    ![Alt text](Images/2.png)

__Step 2: Open port 8000__

* I clicked on instance ID of the unique webservers

* I clicked on security and also clicked on 'Edit inbound rules'.

* I added the new rule (Port 8000) and saved it

    ![Alt text](Images/1.png)

__Step 3: Installing Apache on both webservers 1 and 2__

* On the terminal, i changed directory using the command `cd downloads` and I connected to the instance via the local machine using SSH with the command `ssh -i "webstack_key.pem" ubuntu@ec2-35-175-254-47.compute-1.amazonaws.com` for both webservers individually

    ![Alt text](Images/3.png)


    For webserver2

    ![Alt text](Images/28.png)

* I installed Apache2 on both webservers using the command `sudo apt update -y &&  sudo apt install apache2 -y`

    ![Alt text](Images/4.png)

    For webserver2

    ![Alt text](Images/29.png)

* I verified that Apache2 is running using the command `sudo systemctl status apache2`

    ![Alt text](Images/5.png)

    For webserver2

    ![Alt text](Images/30.png)

__Step 4: Configuring Apache to serve a page showings its public IP__

* I configured Apache2 to serve content on port 8000
     1. I used text editor nano to open the file >ports.conf using the command `sudo nano /etc/apache2/ports.conf`

     ![Alt text](Images/6.png)

        For webserver2

    ![Alt text](Images/31.png)

     2. I added a new listen directive for port 8000 and saved it

        ![Alt text](Images/7.png)

        For webserver2

        ![Alt text](Images/33.png)

     3. I opened the 000-default.conf file, changed port 80 on the virtual host to 8000 using the command `sudo vi /etc/apache2/sites-available/000-default.conf`

        ![Alt text](Images/9.png)

        ![Alt text](Images/8.png)

        For webserver2

        ![Alt text](Images/32.png)

     4. I saved and closed the file 

     5. I restarted apache2 on both servers to load the new configuration using the command `sudo systemctl restart apache2`

        ![Alt text](Images/10.png)

        For webserver2

        ![Alt text](Images/34.png)

* __Creating my html file__

     1. I opened a new html file using the command `sudo nano index.html` 

        ![Alt text](Images/12.png)

        For webserver 2

        ![Alt text](Images/35.png)

     2. I edited the file and tagged the name of the webservers. I also copied the public ip of both webservers and inserted it into the file

        ![Alt text](Images/11.png)

     3. I changed the file ownership with the command `sudo chown www-data:www-data ./index.html`

        ![Alt text](Images/13.png)

* __Overriding the Default html file of Apache webserver:__

    1. I replaced the default html file with the new file using the command `sudo cp -f ./index.html /var/www/html/index.html`

        ![Alt text](Images/14.png)

        For webserver2

        ![Alt text](Images/36.png)


    2. I restarted the webserver to load the new configuration with the command `sudo systemctl restart apache2`

        ![Alt text](Images/15.png)

        For webserver2

        ![Alt text](Images/37.png)

    3. I got this web page result

        ![Alt text](Images/26.png)

        ![Alt text](Images/38.png)

__Step 5: Configuring Nginx as a load balancer__

    * I opened a load balancer EC2 instance and made sure port 80 is opened to accept traffic from anywhere

 ![Alt text](Images/16.png)

    * I SSH into the instance using the command `ssh -i webstack_key.pem ubuntu@107.21.33.182`

 ![Alt text](Images/17.png)

    * I updated the server and installed Nginx using the command `sudo apt update -y && sudo apt install nginx -y`

 ![Alt text](Images/18.png)

    * I verified the installation using the command `sudo systemctl status nginx`

![Alt text](Images/19.png)

    * I opened Nginx configuration file with the command `sudo vi /etc/nginx/conf.d/loadbalancer.conf`

![Alt text](Images/20.png)

    * I pasted the configurated file below, edited and pasted the correct IP of the webservers and the loadbalancers

            
        upstream backend_servers {

            # your are to replace the public IP and Port to that of your webservers
            server 127.0.0.1:8000; # public IP and port for webserser 1
            server 127.0.0.1:8000; # public IP and port for webserver 2

        }

        server {
            listen 80;
            server_name <your load balancer's public IP addres>; # provide your load balancers public IP address

            location / {
                proxy_pass http://backend_servers;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
        }
    
![Alt text](Images/21.png)

    * I used this command to silence the default configuration file `sudo nano nginx.conf`

![Alt text](Images/22.png)    

    * I tested the configuration with the command `sudo nginx -t`

![Alt text](Images/23.png)

    * I restarted Nginx to load the new configuration using the command `sudo systemctl restart nginx`

![Alt text](Images/24.png)

    * I reloaded the web pages

![Alt text](Images/25.png)

![Alt text](Images/27.png)




> END OF PROJECT 
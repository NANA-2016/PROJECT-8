# PROJECT-8

## Automating Load Balancing With nginx

 ## Deploying and configuring webservers.(APACHE2 webservers)

  In this project EC2 instances on AWS are first created  which will later run Apache2 webservers both Listening on port 8000 as demonstrated below.

  ## s1 - First Server  with PUBLIC IP ADDRESS 35.178.199.112

 ![server1 with port 8000](https://github.com/NANA-2016/PROJECT-8/assets/141503408/a9ab6109-35d4-4142-a823-00529bbc1eda)

 ## s2 - Second Server  with PUBLIC IP ADDRESS 35.178.184.234

![s2 with port 8000](https://github.com/NANA-2016/PROJECT-8/assets/141503408/7b3c76d1-a813-448d-b698-51b6f78926f2)

  # Connecting webserver Via Terminal ssh.

 ## ssh -i Darey.io.pem ubuntu@35.178.199.112 for server 1(s1)

 ![ssh s1](https://github.com/NANA-2016/PROJECT-8/assets/141503408/c45badc0-1df0-43ce-aacd-87bce6f32b98)

##  ssh -i Darey.io.pem ubuntu@35.178.184.234 for server 2(s2)

 ![ssh s2](https://github.com/NANA-2016/PROJECT-8/assets/141503408/5ff908b4-bf15-4aec-b1df-9d73e0b4b51f)

  ## Installing Apache2 for both webserver done through the command  "sudo apt install apache2"
  
![apache2 installation s1](https://github.com/NANA-2016/PROJECT-8/assets/141503408/5ec42406-c598-4f53-b04b-009ace1d7346)

![apache2 installation s2](https://github.com/NANA-2016/PROJECT-8/assets/141503408/9bb24d0d-ee9e-4a98-82b3-821268274627)

## The codified script below  has been used to  the deploy the two Apache web servers using the command in a file ./install.sh

"sudo vi install.sh"

.

#!/bin/bash

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

 Permissin changes are also done on both servers to allow execution permission using command

    "sudo chmod +x install.sh"

  Running the script using the command below and expected results on both Servers.
  
    "./install.sh PUBLIC_IP"

  ## The Screen shots below demonstrate the process as demonstrated above and the outcome from the web.

  ## Server1

  ![script, permissions, run script  s1](https://github.com/NANA-2016/PROJECT-8/assets/141503408/cf91290a-14ac-4336-9900-0879ca9af24d)

 ![Screenshot 2023-10-25 221206](https://github.com/NANA-2016/PROJECT-8/assets/141503408/79add5be-3ab4-43ee-b637-0797f452c1fb)
 
## Server2

![script, permissions, run script  s2](https://github.com/NANA-2016/PROJECT-8/assets/141503408/11f7a651-9dbf-4737-9c8e-aac9446f7275)

![s2](https://github.com/NANA-2016/PROJECT-8/assets/141503408/9cfef321-4b17-400a-9ef7-e947c3b10dff)


# Deploying and Configuring  Nginx  load Balancer 

## - setting up Nginx nginx webserver instance to be listening on port 80.

![s3 with port 80](https://github.com/NANA-2016/PROJECT-8/assets/141503408/eaecd0fc-9a08-4f06-a87a-c91f8b1f27f6)

## - Nxt is to ssh the inginx webserver and configuring the loadbalancer . 

ssh -i Darey.io.pem ubuntu@18.170.29.170

 ![ssh s3](https://github.com/NANA-2016/PROJECT-8/assets/141503408/f912e346-c709-4aaf-ace5-a36e1a8dcde2)

 ## - Installing nginx 

  ![installing nginx](https://github.com/NANA-2016/PROJECT-8/assets/141503408/fc631a54-f0bb-4497-a2df-c10bdaf1e8a6)

   ## - Configuring nginx as a load balancer.

    These is done by  creating a file with the script below using the command
    
    "sudo vi nginx.sh"

    
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

 ## - Later on execution permission 
 
 "sudo chmod +x nginx.sh" is added to execute the file and run the script using command ./nginx.sh PUBLIC_IP Webserver-1 Webserver-2

The three commands are demostrated below as highlighred on the screenshot.

 - sudo vi nginx.sh ,sudo chmod +x nginx.sh

![file creation](https://github.com/NANA-2016/PROJECT-8/assets/141503408/5e37084d-d4ef-48a1-8032-ca5c216ed43f)

 - ./nginx.sh PUBLIC_IP Webserver-1 Webserver-2

![running nginx script](https://github.com/NANA-2016/PROJECT-8/assets/141503408/614ed5c9-8f73-482f-aaa8-9e11d4fe7043)

## http://18.170.29.170:80 on web.

![S3 WEBPAGE](https://github.com/NANA-2016/PROJECT-8/assets/141503408/f6d7d3d6-3c45-44a9-b233-af5f151ce5e6)
 









    



  


  


 

 












 

 
 

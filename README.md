# OracleCloudInstanceInitialSetup

1.	Create Oracle Compute Instance: select either oracle Linux or CentOS.
2.	Availability zone and capacity similar like AWS.
3.	Boot volume similar like EBS is a secondary volume to run instance.
4.	Make sure Add SSH keys: Open Putty Gen and create private key and save it with (.ppk). Add public key while SSH key field. Private key is unique and associate with its public key. Like private key is key and public key is locker. Such as to open any locker, unique is requires. If you have more than one instances, then generate each of its private and public keys.
5.	Virtual Cloud Network similar like Private and public network (VPN). Make sure create all private and public subnet with port number. 
6.	Ingress Rules – Add CIDR and protocols like TCP or all.
7.	How to connect: Use MobaXTerm, provide IP address, username: blank, port: 22 , add private key and click connect. Similar way left username: blank on putty also.
8.	Connect from console: sudo ssh opc@public-id-address -i /etc/rsa-private-key.ppk
9.	Setup docker: https://technology.amis.nl/2020/02/06/run-always-free-docker-container-on-oracle-cloud-infrastructure/
Please note: Always run with sudo, if yum not work, install it.
sudo yum-config-manager --enable ol7_addons
sudo yum install docker-engine -y
sudo systemctl start docker
sudo systemctl enable docker

10. Install and connect Alpine on docker:
# docker pull alpine
##### Usually, an Alpine Linux image doesn't contain bash, Instead you can use /bin/ash, /bin/sh, ash or only sh.
##### /bin/ash

docker run -it --rm alpine /bin/ash
/bin/sh
docker run -it --rm alpine /bin/sh
ash
docker run -it --rm alpine ash
sh
docker run -it --rm alpine sh

11. Setup nginx container image with this command:
sudo docker run -d \
--restart always \
--hostname nginx.lucasjellema.com \
-p 3456:443 \
-p 3457:80 \
--name my-nginx \
nginx 

(So nginx is rinning on docker!)

12.	curl http://publicIPAddress:3457

13.	Setup Apache HTTP Server(This is not in docker): https://scriptingmysql.wordpress.com/2019/11/19/oracles-always-free-cloud-instance-adding-a-web-server-to-your-free-mysql-compute-instance-with-zero-monthly-charges/
sudo yum install httpd -y
sudo apachectl start
sudo systemctl enable httpd
sudo apachectl configtest
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd –reload
sudo bash -c 'echo This is my new web page running on Oracle Cloud Always Free compute instance > /var/www/html/index.html'
curl http://publicIPAddress:80
To setup new react application: build the app, rename the project folder to html and replace the html file at the VM location: /var/www/. Thats it!

14. Setup Apache HTTP Server(This is in docker):
sudo docker pull httpd:latest
sudo docker run -dit --name <image name: httpd> -p 8080:80 -v /var/www/html/:/usr/local/apache2/htdocs/ <httpd:2.4 or httpd:latest>
Example: sudo docker run -dit --name httpd -p 8080:80 -v /var/www/html/:/usr/local/apache2/htdocs/ httpd:2.4
curl http://publicIPAddress:8080
We can get an interactive shell session to the container using the 'docker exec' command.
docker exec -ti docker-httpd /bin/bash

To setup new react application: build the app, rename the project folder to html and replace the html file at the VM location: /var/www/. Thats it!

15. Oracle Autonomous Database detail instructions:
https://docs.oracle.com/en/cloud/paas/autonomous-data-warehouse-cloud/tutorial-getting-started-autonomous-db/index.html

16. Oracle Autonomous Data Warehouse detail instructions:
https://docs.oracle.com/en/cloud/paas/autonomous-data-warehouse-cloud/tutorial-getting-started-autonomous-db-adw/index.html


# How to Install Jenkins on CentOS 7

Jenkins is a popular open source CI (Continuous Integration) tool which is widely used for project development, deployment, and automation.
This article will guide you through the process of installing Jenkins on a Vultr CentOS 7 server instance. In order to facilitate visitors' access, Nginx will also be installed as the reverse proxy for Jenkins.
Prerequisites
Before proceeding, you must have:
•	Deployed a Vultr CentOS 7 server instance from scratch.
•	Logged into your machine as a non-root user with sudo privileges.
Step 1: Update your CentOS 7 system
One of the Linux system administrator's best practices is keeping a system up to date. Install the latest stable packages, then reboot.
sudo yum install epel-release
sudo yum update
sudo reboot
When the reboot finishes, login with the same sudo user.
Step 2: Install Java
Before you can install Jenkins, you need to setup a Java virtual machine on your system. Here, let's install the latest OpenJDK Runtime Environment 1.8.0 using YUM:
sudo yum install java-1.8.0-openjdk.x86_64
After the installation, you can confirm it by running the following command:
java -version
This command will tell you about the Java runtime environment that you have installed:
openjdk version "1.8.0_91"
OpenJDK Runtime Environment (build 1.8.0_91-b14)
OpenJDK 64-Bit Server VM (build 25.91-b14, mixed mode)
In order to help Java-based applications locate the Java virtual machine properly, you need to set two environment variables: "JAVA_HOME" and "JRE_HOME".
sudo cp /etc/profile /etc/profile_backup
echo 'export JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk' | sudo tee -a /etc/profile
echo 'export JRE_HOME=/usr/lib/jvm/jre' | sudo tee -a /etc/profile
source /etc/profile
Finally, you can print them for review:
echo $JAVA_HOME
echo $JRE_HOME
Step 3: Install Jenkins
Use the official YUM repo to install the latest stable version of Jenkins, which is 1.651.2 at the time of writing:
cd ~ 
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum install jenkins
Start the Jenkins service and set it to run at boot time:
sudo systemctl start jenkins.service
sudo systemctl enable jenkins.service
In order to allow visitors access to Jenkins, you need to allow inbound traffic on port 8080:
sudo firewall-cmd --zone=public --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
Now, test Jenkins by visiting the following address from your web browser:
http://<your-Vultr-server-IP>:8080
Step 4: Install Nginx (optional)
In order to facilitate visitors' access to Jenkins, you can setup an Nginx reverse proxy for Jenkins, so visitors will no longer need to key in the port number 8080 when accessing your Jenkins application.
Install Nginx using YUM:
sudo yum install nginx
Modify the configuration of Nginx:
sudo vi /etc/nginx/nginx.conf
Find the two lines below:
location / {
}
Insert the six lines below into the { } segment:
proxy_pass http://127.0.0.1:8080;
proxy_redirect off;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
The final result should be:
location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_redirect off;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
Save and quit:
:wq
Start and enable the Nginx service:
sudo systemctl start nginx.service
sudo systemctl enable nginx.service
Allow traffic on port 80:
sudo firewall-cmd --zone=public --permanent --add-service=http
sudo firewall-cmd --reload
Finally, visit the following address from your web browser to confirm your installation:
http://<your-Vultr-server-IP>


Reference: https://www.vultr.com/docs/how-to-install-jenkins-on-centos-7

# Sample comands
sudo lsof -i:8080
sudo kill -9 PID
sudo rm -rf /var/lib/jenkins/
sudo yum install jenkins
sudo yum autoremove jenkins
sudo systemctl start jenkins
sudo systemctl restart jenkins
sudo systemctl enable jenkins
sudo systemctl stop jenkins
sudo lshw
sudo lscpu
ip address
cp myfile.txt myfilecopy.txt

sudo docker ps -a
sudo docker images
sudo docker rm <container id>
sudo docker rmi <image id>
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl stop docker
sudo systemctl status docker


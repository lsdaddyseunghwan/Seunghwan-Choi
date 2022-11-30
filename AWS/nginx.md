# Hands-on Nginx : How to Install Nginx Web Server on EC2 Linux 2

Purpose of the this hands-on training is to give the students basic knowledge of how to install Nginx Web Server on Amazon Linux 2 EC2 instance.

## Learning Outcomes

At the end of the this hands-on training, students will be able to;

- demonstrate their knowledge of how to launch AWS EC2 Instance.

- establish a connection with AWS EC2 Instance with SSH.

- install the Nginx Server on Amazon Linux 2 Instance.

- configure the Nginx Server to run simple HTML page.

- write a simple bash script to run the Web Server

- automate the process of installation and configuration of a Web Server using the `user-data` script of EC2 Instance.

## Outline

- Part 1 - Launching an Amazon Linux 2 EC2 instance and Connect with SSH

- Part 2 - Installing and Configuring Nginx Web Server to Run a Simple Web Page

- Part 3 - Automation of Web Server Installation through Bash Script

## Part 1 - Launching an Amazon Linux 2 EC2 instance and Connect with SSH

-1.  Launch an Amazon 2 (one for spare)EC2 instance with AMI as `Amazon Linux 2`, instance type as `t2.micro` and default VPC security group which allows connections from anywhere and any port.

0. Connect to your instance with SSH.


ssh -i [Your Key pair] ec2-user@[Your EC2 IP / DNS name]


## Part 2 - Installing and Configuring Nginx Web Server to Run a Simple Web Page

1. Update the installed packages and package cache on your instance.


sudo yum update -y


2. Install the Nginx Web Server and Git.

```bash
sudo amazon-linux-extras install nginx1.12 -y
sudo yum install git -y
```

3. Start the Nginx Web Server.

```bash
sudo systemctl status nginx
sudo systemctl start nginx
```

4. Check from browser with public IP/DNS


5. Go to /usr/share/nginx/html folder.

```bash
cd /usr/share/nginx/html
```

6. Show content of folder and change the permissions of /usr/share/nginx/html

```bash
ls
sudo chmod -R 777 /usr/share/nginx/html
```

7. Remove existing `index.html`.

```bash
sudo rm index.html
```

8. Upload new `designer web site` with `git clone` command.

```bash
git clone https://github.com/ethan-techtorial/designer.git
cp -R ./designer/. /usr/share/nginx/html
```

9. restart the Nginx Web Server.

```bash
sudo systemctl restart nginx
```

10. configure to start while launching
```bash
sudo systemctl enable nginx
```

## Part 3 - Automation of Web Server Installation through Bash Script (User data)

11. Configure an Amazon EC2 instance with AMI as `Amazon Linux 2`, instance type as `t2.micro`, default VPC security group which allows connections from anywhere and any port.

12. Configure instance to automate web server installation with `user data` script.


#! /bin/bash
yum update -y
amazon-linux-extras install nginx1.12
yum install git -y
systemctl start nginx
cd /usr/share/nginx/html
git clone https://github.com/ethan-techtorial/designer.git
chmod -R 777 /usr/share/nginx/html
rm index.html
cp -R ./designer/. .
systemctl restart nginx
systemctl enable nginx

13. Review and launch the EC2 Instance

14. Once Instance is on, check if the Nginx Web Server is working from the web browser.

15. Connect the Nginx Web Server from the terminal with `curl` command.


curl http://<IPADRESS>





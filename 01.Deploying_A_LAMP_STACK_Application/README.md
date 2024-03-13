# Deploying a LAMP Stack Web Application on AWS Cloud
***
A LAMP Stack is a solution stack that is used for the deployment of web applications. It stands for Linux, Apache, MySql, and Php,Perl or Python. 

**Linux**: this is an operating system which serves as the backbone of the LAMP Stack, and it is actively used in deploying other components.

**Apache**: is a web server software which via HTTP requests processes requests and transmits information via the internet.

**MySQL**: use for creating and maintaining dynamic databases. It supports SQL and relational tables and provides a DBMS(Database Management System).

**PHP,PERL or PYTHON**: this represents programming languages which effectively combines all the elements of the LAMP stack and is used to make web applications execute.

## Creation of EC2 Instance
First we log on to AWS Cloud Services and create an EC2 Ubuntu VM instance. When creating an instance, choose keypair authentication and `download private key(*.pem)` on your local computer.

On the terminal, `cd` into the directory containing the downloaded private key. 

Run the below command to log into the instance via ssh:

`ssh -i <private_keyfile.pem> username@ip-address`

Successful login into ec2 instance:
![Successful login into instance](./img/1.logged_into_ec2_instance.png)

## Setting Up Apache Web Server

To deploy the web application, we need to install apache via ubuntu package manager `apt`:
```
# Updating Packages
sudo apt update

# Install apache
sudo apt install apache2 -y

```

To verify that apache2 is running as a Service in our OS, run the following command

```
$ sudo systemctl status apache2
```
![Apache server running](./img/2.apache_running.png)

Run the below command to ensure apache2 starts automatically on system boot
```
systemctl enable apache2
```


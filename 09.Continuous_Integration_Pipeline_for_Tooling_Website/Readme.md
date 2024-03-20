## Tooling Website deployment automation with Continuous Integration

## Introduction
Jenkins is an open-source `Continuous Integration` server written in Java for orchestrating a chain of actions to achieve the Continuous Integration process in an automated fashion. Jenkins supports the complete development life cycle of software from building, testing, documenting the software, deploying, and other stages of the software development life cycle.

![jenkins_architecture](./img/jenkins-continuous-integration-min.png)

Here I have enhanced the architecture prepared in Project 8 by adding a Jenkins server, and also configured a job to automatically deploy source code changes from Git to NFS server.

## Jenkins Web Architecture For CI Builds
 
![my_jenkins_architecture](./img/2022-10-05_050932.png)

## Installing Jenkins Server
Spin up an Ubuntu 20.04 server on AWS cloud and SSH into it. Name it `Jenkins`.

Install JDK which is an important Java based package required for Jenkins to run.
```
sudo apt update
sudo apt install default-jdk-headless -y
```
Install Jenkins
```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y

sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```
![jenkins_installation](./img/1.jenkins_running.png)

Since Jenkins runs on default port 8080, open this port on the Security Group inbound rule of the jenkins server on AWS 

Jenkins is up and running, copy and paste jenkins server public ip address appended with port 8080 on a web server to gain access to the interactive console. `<jenkins_server_public_ip_address>:8080`

![jenkins_install_success](./img/2.jenkins_initial_setup.png)

The admin password can be found in the `'/var/lib/jenkins/secrets/initialAdminPassword'` path on the server.

Choose suggested plugins when asked which plugings to install 

Once plugins installation is done – create an `admin user` and we will get the Jenkins server address.

![login_success](./img/3.jenkins_login.png)

## Attaching WebHook to Jenkins Server

On the github repository that contains application code, create a webhook to connect to the jenkins job. To create webhook, go to the settings tab on the github repo and click on webhooks.
Webhook should look like this `<public_ip_of_jenkins_server>:8080/github-webhook/`

![webhook_creation](./img/4.webhook_creation.png)

## Creating Job and Configuring GIT Based Push Trigger

Jenkins web console, click `New Item` and create a `Freestyle project`

In configuration of the Jenkins freestyle job choose Git repository, provide there the link to the `tooling` GitHub repository and credentials (user/password) so Jenkins could access files in the repository. Also specify the branch containing code

![specify_credentials](./img/5.jenkins_job_config.png)

Save configuration and to run the build manually. Click `Build Now` button and should be under #1

![build_success](./img/6.build_job_1.png)

You can open the build and check in `Console Output` if it has run successfully.

![console_output](./img/7.console_output.png)

## Configuring Build Triggers

This build does not produce anything and it runs only when we trigger it manually. To fix this:

- Click `Configure` your job/project and add these two configurations

### 1. Configure triggering the job from GitHub webhook:

![adding_build_triggers](./img/8.build_trigger.png)

### 2. Configure `Post-build Actions` to archive all the files – files resulted from a build are called `artifacts`.

![post_build_step](./img/9.post_build_action.png)

Now, we can go ahead and make some change in any file in our `tooling` GitHub repository (e.g. Readme.Md file) and push the changes to the `master` branch.

A new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server

![build_3](./img/10.github_build_3.png)

![build_3_console_outpu](./img/11.build_3_console.png)

An automated Jenkins job that receives files from GitHub by webhook trigger has been configured(this method is considered as `push` because the changes are being `pushed` and files transfer is initiated by GitHub)

By default, the artifacts are stored on Jenkins server locally
```
ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
```

![git_push](./img/12.build_3_archive.png)

## Configuring Jenkins To Copy Files(Artifact) to NFS Server

To achieve this, we install the `Publish Via SSH` pluging on Jenkins.
The plugin allows one to send newly created packages to a remote server and install them, start and stop services that the build may depend on and many other use cases.

On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item.

On "Available" tab search for "Publish Over SSH" plugin and install it
![plugin](./img/13.publish_ssh_plugin.png)

Configure the job to copy artifacts over to NFS server.
On main dashboard select `Manage Jenkins` and choose "Configure System" menu item.

Scroll down to `Publish over SSH` plugin configuration section and configure it to be able to connect to the NFS server:

Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)

Hostname – can be `private IP address` of NFS server
Username – ec2-user (since NFS server is based on EC2 with RHEL 8) 
Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server

Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.

![setting_publish](./img/14.pub_ssh_config.png)
![server_add](./img/15.test_pub_config.png)

Save the configuration, open your Jenkins job/project configuration page and add another `Post-build Action`

We specify `**` on the `send build artifacts` tab meaning it sends all artifact to specified destination path(NFS Server). 

![ssh_transfer_config](./img/16.ssh_server_transfer.png)

Now make a new change on the source code and push to github, Jenkins builds an artifact by downloading the code into its workspace based on the latest commit and via SSH it publishes the artifact into the NFS Server to update the source code. 

This is seen by the change of name on the web application

![](./img/17.index_php_change.png)

![](./img/18.updated_website.png)

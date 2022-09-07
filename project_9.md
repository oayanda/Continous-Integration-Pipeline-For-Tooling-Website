# INSTALL AND CONFIGURE JENKINS SERVER

Lunch an AWS EC2 instance server based on Ubuntu Server 20.04 LTS and name it Jenkins.
![Aws EC2 Jenkins Instance](./images/1.png)

Login into the terminal and install JDK

```bash
sudo apt update -y && sudo apt install default-jdk-headless -y
```

![Aws EC2 Jenkins Instance](./images/2.png)

Install Jenkins

```bash
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```

![Aws EC2 Jenkins Instance](./images/3.png)

Verify Jenkins up and running

```bash
systemctl status jenkins
```

![Aws EC2 Jenkins Instance](./images/4.png)

By default Jenkins server uses TCP port 8080 – Let's open it by creating a new Inbound Rule in your EC2 Security Group

![Aws EC2 Jenkins Instance](./images/5.png)

Let's setup Jenkins - from the broswer, use the jenkins instance public Ip:8080

![Aws EC2 Jenkins Instance](./images/6.png)

To get the administrator's password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

![Aws EC2 Jenkins Instance](./images/7.png)

Enter the password and click on continue

![Aws EC2 Jenkins Instance](./images/8.png)

Click on the ```install suggested plugins``` , once installation is completed, you would be required to create an admin account aand you will get your Jenkins server address.
![Aws EC2 Jenkins Instance](./images/10.png)

Jenkins setp is complete
![Aws EC2 Jenkins Instance](./images/11.png)

## Configure Jenkins to retrieve source codes from GitHub using Webhooks to be stored locally in Jenkins.

Time to enable Webhook in my GitHub repo.
Use the following steps: 
Go to the repo you want to store in Jenkins - my case the tooling repo. Click
```settings > Webhooks > Add webhook```
> In the ```Payload URL field```, insert the jenkins url (public ip) and the github-webhook path like this
```http://3.84.150.252:8080/github-webhook```

> In the Content type field, select ```application/json```

> For - Which events would you like to trigger this webhook? Make sure, ```Just the push event``` is selected and click Add webhook. See details below.

![Aws EC2 Jenkins Instance](./images/12.png)

Next, Go to Jenkins web console, click "New Item" and create a ```Freestyle project```
Give the job/project a name ```tooling-github``` and click ok.
![Aws EC2 Jenkins Instance](./images/14.png)

To connect my GitHub repository, I need to provide its URL, this was copy it from the repository itself

![Aws EC2 Jenkins Instance](./images/15.png)

In the Source Code Management tab of the Jenkins freestyle project choose Git repository, provide there the link to the Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.
![Aws EC2 Jenkins Instance](./images/16.png)
Now, under Credentials, select the newly created github login details and click save.
![Aws EC2 Jenkins Instance](./images/17.png)
For now I will run the build manually.
Click on ```Build Now```
![Aws EC2 Jenkins Instance](./images/18.png)
The green mark ```#1``` show the build was successful.
![Aws EC2 Jenkins Instance](./images/19.png)
The above build was ran manually, now I will configure the job to use the earlier setup webhook.

In the project dashboard, Click "Configure" your job/project and add these two configurations
![Aws EC2 Jenkins Instance](./images/20.png)
Click on ```Build Triggers``` and select ```GitHub hook trigger for GITScm polling``` to Configure triggering the job from GitHub webhook
![Aws EC2 Jenkins Instance](./images/21.png)
Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts". Click on ```Add post-build action``` and select ```Archive the artifacts```. In the  ```Files to archive?```  input  ```**``` to archive mulitiple nested artifacts and click on save.
![Aws EC2 Jenkins Instance](./images/22.png)

Now let's verify the configuration, I will edit the README.md to make some changes.
![Aws EC2 Jenkins Instance](./images/23.png)
You can build ```#2```was done automatically with the webhook setup
![Aws EC2 Jenkins Instance](./images/24.png)
for more details, click build ```#2``` and check the console output
![Aws EC2 Jenkins Instance](./images/25.png)

By default, the artifacts are stored on Jenkins server locally

```bash
ls /var/lib/jenkins/jobs/tooling-github/builds/2/archive/
```
![Aws EC2 Jenkins Instance](./images/26.png)

## CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH

Currently, the artifacts is stored in Jenkins but in order to implement a Jenkins CI it needs to be in the NFS Server to serve data to the Web Servers - by so doing, any changes made in the version control system (GitHub) would automatically update the Web Servers.

The next step is to copy them to the NFS server ```/mnt/apps``` directory.

To do this securely in Jenkins, I would install a plugin called ```Publish Over SSH```

On the main dashboard click ```Manage Jenkins``` and select ```Manage Plugins```

![Aws EC2 Jenkins Instance](./images/27.png)

Click on Available and search for ```Publish Over SSH``` select and install without restart
![Aws EC2 Jenkins Instance](./images/28.png)

Now, let's Configure the job/project to copy artifacts over to NFS server.

On main dashboard select ```Manage Jenkins``` and choose ```Configure System``` menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server

1.  Provide a private key (content of .pem file that you use to connect to NFS server via SSH).

*Add the Passphrase if the private key is configured with one.*

2. Click on ```SSH Servers``` to expand and file the required details, Test configuration and save.

> Hostname – can be private IP address of your NFS server

> Username – ec2-user (since NFS server is based on EC2 with RHEL 8)

> Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server.

![Aws EC2 Jenkins Instance](./images/29.png)

![Aws EC2 Jenkins Instance](./images/30.png)

open Jenkins job/project configuration page and add another one ```Post-build Action```

Click on ```Add post-build action``` and select ```Send build artifacts over SSH```
Configure it to send all ```( denoted by ** )``` files probuced by the build into our previouslys define remote directory.

Save this configuration 

![Aws EC2 Jenkins Instance](./images/31.png)
Now, lets verify - change something in README.MD file in the GitHub Tooling repository.
![Aws EC2 Jenkins Instance](./images/32.png)

Login into the NFS Server instance and change the permission to allow Jenkins to save files in the /mnt directory.
![Aws EC2 Jenkins Instance](./images/33.png)

Verify a successful and automically build in Jenkins
![Aws EC2 Jenkins Instance](./images/34.png)


Verify the files was succesfully updated in ```/mnt/apps``` 
![Aws EC2 Jenkins Instance](./images/35.png)

Check the changes in README.md file
![Aws EC2 Jenkins Instance](./images/36.png)

✨✔️ ✔️
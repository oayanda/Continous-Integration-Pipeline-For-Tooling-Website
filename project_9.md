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

Configure Jenkins to retrieve source codes from GitHub using Webhooks


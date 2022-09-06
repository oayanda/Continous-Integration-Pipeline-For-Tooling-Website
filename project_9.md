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
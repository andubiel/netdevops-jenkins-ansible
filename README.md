# netdevops-ansible CICD Lab
This lab can run "as is" using the following dcloud pod https://dcloud.cisco.com/
Search for Devnet Express DNAv3

# VPN to POD
Select "Info" for credentials and use Anyconect or OpenVPN to connect to POD wih SSL VPN.
Info:

![Picture1](https://user-images.githubusercontent.com/11307137/55585605-11602280-56f5-11e9-99ab-dbdc2ee688b9.png)

![Picture2](https://user-images.githubusercontent.com/11307137/55585971-df02f500-56f5-11e9-9bf3-aaf6b224f885.png)

Centos 198.18.134.48 is used as our Dev Box for this lab, You can 
```
$ ssh developer@198.18.134.48  pass = C1sco12345 or Remote Desktop from the following link to remotely access.
```

![Picture3](https://user-images.githubusercontent.com/11307137/55586148-3ef99b80-56f6-11e9-80e6-3e99e98388d5.png)

# Setup the Centos "DevBox"
Carefull, If cut and pasting try to do one or two lines at a time and verify ok. Scripting the following would make more sense but we would like you to see the steps in action for the first run through.  
```
ls
code  Desktop  dev_box_background.jpg  Downloads  sync_log.txt  sync_version  thinclient_drives
cd code
git clone https://github.com/andubiel/netdevops-ansible.git
cd netdevops-ansible/
sudo pip install -r requirements.txt
sudo systemctl disable firewalld
sudo systemctl stop firewalld
```
# Install GOGs
Install docker container for Go for Git (GOGs). This will provide a Github like repository for your lab environment.
```
[developer@centos netdevops-ansible]$ docker volume create --name gogs-data
docker run --name=gogs -p 10022:22 -p 3000:3000 -v gogs-data:/data gogs/gogs

When finished type $ ctrl c to stop terminal and then start the Docker container.
docker start gogs
```
Setup GOGs from the web GUI. http://198.18.134.48:3000

Change Database to sqlite3.
![Picture4](https://user-images.githubusercontent.com/11307137/55587330-1c1cb680-56f9-11e9-949c-867728d5f92d.png)

Change GOGs URL to 198.18 134.48/3000 if not already displaying and then start GOGs.

![Picture5](https://user-images.githubusercontent.com/11307137/55587483-6ef66e00-56f9-11e9-9ece-5dd2d486ef59.png)

Next sign-up for a local account on the GOGs server. Use netdevopsuser for user and network as the password.
Note the following information for later:
```
git config --global user.email netdevopsuser@netdevops.local
git config --global user.name netdevopsuser
```
![Picture6](https://user-images.githubusercontent.com/11307137/55587694-eaf0b600-56f9-11e9-91fa-dd4b180f858f.png)

Login with user netdevopsuser and password network

![Picture7](https://user-images.githubusercontent.com/11307137/55587755-170c3700-56fa-11e9-82ef-5374eb84c947.png)

Now add the first repository. 

![Picture8](https://user-images.githubusercontent.com/11307137/55588739-96026f00-56fc-11e9-8645-daac5bfbfbb6.png)

![Picture9](https://user-images.githubusercontent.com/11307137/55588833-ccd88500-56fc-11e9-92b7-58755f3bdb0d.png)

Remove git from netdevops-ansible directory to remove the cloned GIT configuration fro GITHUB that was pulled down and start/initiialize a new repository with the GOGs repo.

Try to be conservitive on the cut and paste. :)

```
[developer@centos netdevops-ansible]$ rm -rf .git
touch README.md
git init
git config --global user.email netdevopsuser@netdevopslocal
git config --global user.name netdevopsuser
git add README.md
git commit -m "first commit"
git remote add origin http://198.18.134.48:3000/netdevopsuser/netdevops-ansible.git
git config --global push.default matching
git add --all
git commit -m "files"
git push -u origin master
Username for 'http://198.18.134.48:3000': netdevopsuser
Password for 'http://netdevopsuser@198.18.134.48:3000': network
#Output * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
```
Validate that the files are now viewable from GoGs.
http://198.18.134.48:3000/netdevopsuser/netdevops-ansible

![Picture10](https://user-images.githubusercontent.com/11307137/55589203-ac5cfa80-56fd-11e9-8010-b85e47527e89.png)

Now lets start a new branch to make changes to the files in a safe sandbox.
```
[developer@centos netdevops-ansible]$git checkout -b dev
#Output Switched to a new branch 'dev'
git push -u origin dev
#Output Total 0 (delta 0), reused 0 (delta 0)

Username for 'http://198.18.134.48:3000': netdevopsuser
Password for 'http://netdevopsuser@198.18.134.48:3000': network

To http://198.18.134.48:3000/netdevopsuser/netdevops-ansible.git
 #Output * [new branch]      dev -> dev
Branch dev set up to track remote branch dev from origin.
```
Verify there is now a dev branch on the GOGs repo.

![Picture11](https://user-images.githubusercontent.com/11307137/55589385-183f6300-56fe-11e9-8841-fa552047e7bb.png)

![Picture12](https://user-images.githubusercontent.com/11307137/55589439-360cc800-56fe-11e9-9cc4-6be8ea45f212.png)

# Install Jenkins 
We will install Jenkins directly to the Centos "DevBox".

1st we need to install some dependencies with Java, Tomcat and Jenkins 

```
[developer@centos]$ sudo yum install java-1.8.0-openjdk
sudo yum install wget -y
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
sudo yum install Jenkins -y
[developer@centos ~]$sudo service jenkins start
```
Now use the url  http://198.18.134.48:8080 to setup Jenkins.

![Picture13](https://user-images.githubusercontent.com/11307137/55589638-bc290e80-56fe-11e9-8dfd-45a0b251261f.png)

Use Terminal to access your temporary admin password.
```
[developer@centos ~]$sudo more /var/lib/jenkins/secrets/initialAdminPassword
#Output Copy ---> 51d0d1f031ae48fbac3dec8f27ccc917

```
Install recommended plugins.
![Picture14](https://user-images.githubusercontent.com/11307137/55590062-e0d1b600-56ff-11e9-9f98-ec63dbf3ec4d.png)

Username= netdevopsuser
Pass=network
Email= netdevopsuser@netdevops.local
Save an continue

![Picture15](https://user-images.githubusercontent.com/11307137/55590132-0bbc0a00-5700-11e9-8594-129bba34fc47.png)

Save and Finish and Start using Jenkins.

![Picture16](https://user-images.githubusercontent.com/11307137/55590237-4d4cb500-5700-11e9-8f25-d5155fa4b356.png)

Goto website http://198.18.134.48:8080 
Login and click manage jenkins

![Picture17](https://user-images.githubusercontent.com/11307137/55590285-6eada100-5700-11e9-9b43-ce31f82eecd5.png)

Click manage plugins

![Picture18](https://user-images.githubusercontent.com/11307137/55590344-98ff5e80-5700-11e9-845e-0ad661494eaf.png)

From the available plugins find and install without restart the gogs, ansible, and Cisco Spark plugins.

![Picture19](https://user-images.githubusercontent.com/11307137/55590431-e1b71780-5700-11e9-9639-7a09915b775d.png)

![Picture20](https://user-images.githubusercontent.com/11307137/55590475-feebe600-5700-11e9-88c2-aa2426d3edbe.png)

Create a new Job from new Item.

![Picture21](https://user-images.githubusercontent.com/11307137/55590545-30fd4800-5701-11e9-9e25-8db596d0f255.png)

Configure a new Pipeline job, and click OK.

![Picture22](https://user-images.githubusercontent.com/11307137/55590641-74f04d00-5701-11e9-9f12-c1e0738ecac8.png)

Scroll through to make the following configuration selections and click save.

![Picture23](https://user-images.githubusercontent.com/11307137/55590698-a0733780-5701-11e9-88cb-5bc28e3013ea.png)
![Picture24](https://user-images.githubusercontent.com/11307137/55590734-baad1580-5701-11e9-85a5-afd6ab52d9f8.png)

Repository URL: http://198.18.134.48:3000/netdevopsuser/netdevops-ansible.git
Repository Browser = GOGs

# Cisco Webex Teams "Spark"

Spark credentials. We need to convert our Spark Bot Token created earlier to a Jenkins credential.
1st Select Credentials:

![Picture25](https://user-images.githubusercontent.com/11307137/55590982-6f473700-5702-11e9-9547-76c94db65d05.png)

Global Credential, click add credentials:
![Picture26](https://user-images.githubusercontent.com/11307137/55591060-97cf3100-5702-11e9-8da6-f1a7ae67e595.png)

You nw need to paste in your Spark bot token saved earlier and click ok to view the ID which becomes the credentialsId we use for spark in the Pipeline:

![Picture27](https://user-images.githubusercontent.com/11307137/55591139-c3eab200-5702-11e9-8fe5-116a6524ace2.png)
![Picture28](https://user-images.githubusercontent.com/11307137/55591175-d95fdc00-5702-11e9-9bf5-1248d5d552ae.png)

Save your Sparkbot ID

# GOGs and Jenkins Integration
At this point we can prepare the Gogs server to integrate with the Jenkins server. Click on settings.

![Picture29](https://user-images.githubusercontent.com/11307137/55593153-d962da80-5708-11e9-9440-6ca987aad600.png)

Click webhooks:

![Picture30](https://user-images.githubusercontent.com/11307137/55593198-01523e00-5709-11e9-96b1-c2756eba1a7d.png)

Add webhook + Gogs:

![Picture31](https://user-images.githubusercontent.com/11307137/55595701-20ee6400-5713-11e9-8685-80422eda138c.png)

Add webhook:

![Picture32](https://user-images.githubusercontent.com/11307137/55595779-61e67880-5713-11e9-8222-e5830ba98e55.png)






















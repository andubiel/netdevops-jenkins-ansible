# netdevops-ansible CICD Lab
This lab can run "as is" using the following dcloud pod https://dcloud.cisco.com/
Search for Devnet Express DNAv3. Pods in dcloud are reservable for several days.

# The Completed Lab
Upon completion of this lab, your NetDevOps CICD Pipeline will resemble the following.

<img width="835" alt="Screenshot 2019-04-04 20 32 10" src="https://user-images.githubusercontent.com/11307137/55596857-c7893380-5718-11e9-9ec5-4ae3900c9b1a.png">

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
Save and continue

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
In a new tab, navigate to https://developer.webex.com and login. If you do NOT have a Webex Teams account, sign up for a free account.
Click on your avatar in the upper right corner, and select "My Webex Teams Apps". 
Now click the "Plus" symbol in a green circle.

![spark_bot2](https://user-images.githubusercontent.com/11307137/55596038-af171a00-5714-11e9-9454-70aeac340e03.jpg)

Create your bot with the following details.

Name: NetDevOps CICD Bot
Bot Username: Any available name, indicated by green check
Icon: Pick a default choice
Description: DevNet CICD Lab Bot

![spark_bot3](https://user-images.githubusercontent.com/11307137/55596070-d66de700-5714-11e9-8465-c0ea1016030a.jpg)

After clicking "Create Bot", you'll be presented with the Bot Details. At the bottom of the details is the "Bot's Access Token". Click the button to "Copy" the token.  Make sure to note the Token and not mix up with BOT ID.

![spark_bot4](https://user-images.githubusercontent.com/11307137/55596140-277ddb00-5715-11e9-91ec-cb240df85e4e.jpg)

Paste token into a text file for safe keeping!

Next we need to search developer.webex.com to glean the RoomId from our bot. From Room scroll down until you find the Bot RoomId.

<img width="955" alt="Screenshot 2019-04-04 23 07 01" src="https://user-images.githubusercontent.com/11307137/55601328-640a0080-572e-11e9-925c-9b19faf40970.png">

Next we need to search developer.webex.com to glean the RoomId from our bot. From Room run GET v1/rooms/ and scroll down until you find the Bot RoomId. Copy the ID

<img width="500" alt="Screenshot 2019-04-04 23 11 04" src="https://user-images.githubusercontent.com/11307137/55601446-f1e5eb80-572e-11e9-8d8d-24bc85226155.png">

Spark credentials. We need to convert our Spark Bot Token created earlier into a Jenkins credential.
1st Select Credentials from the left panel on the main Jenkins dashboard:

![Picture25](https://user-images.githubusercontent.com/11307137/55590982-6f473700-5702-11e9-9547-76c94db65d05.png)

Under Stores scoped to Jenkins click the Global Credential, and then click add credentials to the left:

![Picture26](https://user-images.githubusercontent.com/11307137/55591060-97cf3100-5702-11e9-8da6-f1a7ae67e595.png)

You now need to paste in your Spark bot token saved earlier and click ok to view the ID which becomes the credentialsId we use for spark in the Pipeline:

![Picture27](https://user-images.githubusercontent.com/11307137/55591139-c3eab200-5702-11e9-8fe5-116a6524ace2.png)
![Picture28](https://user-images.githubusercontent.com/11307137/55591175-d95fdc00-5702-11e9-9bf5-1248d5d552ae.png)

Save your Sparkbot ID it will be used later.

From an editor (VIM, ATOM, Notebook++ etc) modifiy the Dev Notification to include your Sparkbot data
```
[developer@centos netdevops-ansible]$cd ansible-04-mission/
[developer@centos ansible-04-mission]$ls
04-lab_cleanup.yaml  netdevops_dev.yaml   notification-dev.retry  notification-prod.yaml
netdevops_dev.retry  netdevops_prod.yaml  notification-dev.yaml
vim notification-dev.yaml
- name: Cisco Spark - Text Message to a Room
    cisco_spark:
      recipient_type: roomId
      recipient_id: "Your BOT Room ID"
      message_type: text
      personal_token: "Your BOT Token"
      message: "The Dev Build was Successful - Committing Dev to Master"
esc
:wq!
```
From an editor (VIM, ATOM, Notebook++ etc) modifiy the Prod Notification to include your Sparkbot data
```
[developer@centos netdevops-ansible]$cd ansible-04-mission/
[developer@centos ansible-04-mission]$ls
04-lab_cleanup.yaml  netdevops_dev.yaml   notification-dev.retry  notification-prod.yaml
netdevops_dev.retry  netdevops_prod.yaml  notification-dev.yaml
vim notification-prod.yaml
- name: Cisco Spark - Text Message to a Room
    cisco_spark:
      recipient_type: roomId
      recipient_id: "Your BOT Room ID"
      message_type: text
      personal_token: "Your BOT Token"
      message: "The Dev Build was Successful - Committing Dev to Master"
esc
:wq!
```
CD back to netdevops-ansible and edit Jenkins file to edit Notifications in pipeline.

```
[developer@centos ansible-04-mission]$cd ..
[developer@centos netdevops-ansible]$ls
1  12  ansible-04-mission  ansible.cfg  group_vars  hosts  Jenkinsfile  README.md  requirements.txt
vim Jenkinsfile

  stage ('Notification Details') {
            sparkSend credentialsId: 'Change to credential Id created in Jenkins', message: '[$JOB_NAME]($BUILD_URL)', messageType: 'text', spaceList: [[spaceId: ' Change to your Sparkbot Room ID', spaceName: 'NetDevOps CICD Bot']]
         }
    } catch (err) {
        currentBuild.result = 'FAILED'
esc
:wq!

```
Commit changes to GIT

```
[developer@centos ansible-04-mission]$git add notification-dev.yaml
$git add notification-prod.yaml

[developer@centos netdevops-ansible]$git add Jenkinsfile
[developer@centos netdevops-ansible]$git commit -m "notifications"
[dev e8552d9] notifications
 3 files changed, 5 insertions(+), 5 deletions(-)
[developer@centos netdevops-ansible]$git push
Counting objects: 11, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (6/6), 714 bytes | 0 bytes/s, done.
Total 6 (delta 5), reused 0 (delta 0)
Username for 'http://198.18.134.48:3000': netdevopsuser
Password for 'http://netdevopsuser@198.18.134.48:3000':
To http://198.18.134.48:3000/netdevopsuser/netdevops-ansible.git
   4380055..e8552d9  dev -> dev
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'http://198.18.134.48:3000/netdevopsuser/netdevops-ansible.git'
hint: Updates were rejected because a pushed branch tip is behind its remote
hint: counterpart. Check out this branch and merge the remote changes

```

# GOGs and Jenkins Integration
At this point we can prepare the Gogs server to integrate with the Jenkins server. Click on settings.

![Picture29](https://user-images.githubusercontent.com/11307137/55593153-d962da80-5708-11e9-9440-6ca987aad600.png)

Click webhooks:

![Picture30](https://user-images.githubusercontent.com/11307137/55593198-01523e00-5709-11e9-96b1-c2756eba1a7d.png)

Add webhook + Gogs:

![Picture31](https://user-images.githubusercontent.com/11307137/55595701-20ee6400-5713-11e9-8685-80422eda138c.png)

Add webhook:

![Picture32](https://user-images.githubusercontent.com/11307137/55595779-61e67880-5713-11e9-8222-e5830ba98e55.png)

# Run Pipeline and review results
Make sure the pipeline runs successfully by reviwing the console output and Cisco Spark Bot by receiving notifications. In order to run the pipeline we need to initiate the pipeline by making a change to code in our GOGs repository. The Jenkinfile will kickoff the pipeline when dedecting changes. We will add a comment line to the Jenkinsfile and then commit our change to GIT to kickoff the pipeline. As an alterntive you can simply select Build Now from the Jenkins GUI to kickoff a pipeline.

As an alterntive you can select Build Now from Jenkins to kickoff pipeline.
Check Build.

```
[developer@centos netdevops-ansible]$ vim Jenkinsfile

node {
// Added Comment file here
// Clean workspace before doing anything
esc 
:wq!

git add Jenkinsfile
git commit -m Jenkinsfile
git push 
Username for 'http://198.18.134.48:3000': netdevopsuser
Password for 'http://netdevopsuser@198.18.134.48:3000':
```
Check Build

<img width="671" alt="Screenshot 2019-04-04 23 17 38" src="https://user-images.githubusercontent.com/11307137/55601658-fb238800-572f-11e9-9d5a-8df1299fe23f.png">

Open Console

<img width="948" alt="Screenshot 2019-04-04 23 20 21" src="https://user-images.githubusercontent.com/11307137/55601733-4178e700-5730-11e9-93d1-cd4d8584ed1d.png">

Check Webex Team Space "Spark". You will see three sets of notifications. The first set refers to the Dev code changes, while the second set refers to Production and the third verifies the dev branch  was sucesseffully merged to master.

<img width="585" alt="Screenshot 2019-04-04 23 48 12" src="https://user-images.githubusercontent.com/11307137/55602532-214b2700-5734-11e9-8f0b-2d4d0f574693.png">





























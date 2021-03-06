# netdevops-ansible CICD Lab with Jenkins
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
sudo pip install --upgrade setuptools
git clone https://github.com/andubiel/netdevops-jenkins-ansible.git
cd netdevops-jenkins-ansible/
sudo pip install -r requirements.txt

```
# Install GOGs
Install docker container for Go for Git (GOGs). This will provide a Github like repository for your lab environment.
```
[developer@centos netdevops-ansible]$ sudo docker volume create --name gogs-data
sudo docker run --name=gogs -p 10022:22 -p 3000:3000 -v gogs-data:/data gogs/gogs

When finished type $ ctrl c to stop terminal and then start the Docker container.
sudo docker start gogs
```
Setup GOGs from the web GUI. http://198.18.134.48:3000

Change Database to sqlite3.
![Picture4](https://user-images.githubusercontent.com/11307137/55587330-1c1cb680-56f9-11e9-949c-867728d5f92d.png)

Change GOGs Domain to 198.18.134.48 and URL to 198.18.134.48/3000 if not already displaying and then click install GOGs.

<img width="1160" alt="Screenshot 2019-04-11 22 43 30" src="https://user-images.githubusercontent.com/11307137/56010964-43502680-5cab-11e9-9a1d-fb630664ed52.png">

Next sign-up for a local account on the GOGs server. Use netdevopsuser for user and network as the password.
Note the following information for later:
```
user.email netdevopsuser@netdevops.local
user.name netdevopsuser
password = network
```
First you need to setup local account on this local GOGs server. Click signup now and use the above info.

![Picture6](https://user-images.githubusercontent.com/11307137/55587694-eaf0b600-56f9-11e9-91fa-dd4b180f858f.png)

Login with user netdevopsuser and password network

![Picture7](https://user-images.githubusercontent.com/11307137/55587755-170c3700-56fa-11e9-82ef-5374eb84c947.png)

Now add the first repository. 

<img width="616" alt="Screenshot 2019-04-05 13 50 43" src="https://user-images.githubusercontent.com/11307137/55646693-da981400-57a9-11e9-9c89-3f9daf67b5e3.png">

Remove git from netdevops-jenkins-ansible directory to remove the cloned GIT configuration fro GITHUB that was pulled down and start/initiialize a new repository with the GOGs repo.

```
[developer@centos netdevops-jenkins-ansible]$ rm -rf .git
rm README.md
touch README.md
git init
git config --global user.email netdevopsuser@netdevopslocal
git config --global user.name netdevopsuser
git add README.md
git commit -m "first commit"
git remote add origin http://198.18.134.48:3000/netdevopsuser/netdevops-jenkins-ansible.git
git config --global push.default matching
git add --all
git commit -m "files"
git push -u origin master
#Output: Username for 'http://198.18.134.48:3000': netdevopsuser
#Output: Password for 'http://netdevopsuser@198.18.134.48:3000': network
#Output: * [new branch]      master -> master
#Output: Branch master set up to track remote branch master from origin.
```
Validate that the files are now viewable from GoGs.
http://198.18.134.48:3000/netdevopsuser/netdevops-jenkins-ansible

<img width="1373" alt="Screenshot 2019-04-11 22 55 44" src="https://user-images.githubusercontent.com/11307137/56011381-f8cfa980-5cac-11e9-9f7a-43595c7f3c4c.png">

Now lets start a new branch to make changes to the files in a safe sandbox.
```
[developer@centos netdevops-jenkins-ansible]$git checkout -b dev
#Output Switched to a new branch 'dev'
git push -u origin dev
#Output Total 0 (delta 0), reused 0 (delta 0)

Username for 'http://198.18.134.48:3000': netdevopsuser
Password for 'http://netdevopsuser@198.18.134.48:3000': network

To http://198.18.134.48:3000/netdevopsuser/netdevops-jenkins-ansible.git
 #Output * [new branch]      dev -> dev
Branch dev set up to track remote branch dev from origin.
```
Verify there is now a dev branch on the GOGs repo. Refresh you browser if needed.

<img width="1136" alt="Screenshot 2019-04-11 22 58 17" src="https://user-images.githubusercontent.com/11307137/56011472-62e84e80-5cad-11e9-9052-b2dfa64301ff.png">

<img width="1266" alt="Screenshot 2019-04-11 22 59 51" src="https://user-images.githubusercontent.com/11307137/56011502-8d3a0c00-5cad-11e9-8a56-452059c61d61.png">

# Install Jenkins 
We will install Jenkins directly to the Centos "DevBox".

1st we need to install some dependencies with Java, Tomcat and Jenkins 

```
[developer@centos]$ sudo yum install java-1.8.0-openjdk -y 
sudo yum install wget -y
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
sudo systemctl stop firewalld
sudo systemctl disable firewalld
sudo yum install jenkins -y
sudo service jenkins start
```
Now use the url  http://198.18.134.48:8080 to setup Jenkins.

![Picture13](https://user-images.githubusercontent.com/11307137/55589638-bc290e80-56fe-11e9-8dfd-45a0b251261f.png)

Use Terminal and more commad to view your temporary admin password.
```
[developer@centos ~]$sudo more /var/lib/jenkins/secrets/initialAdminPassword
#Output Example "Copy your Key" ---> 51d0d1f031ae48fbac3dec8f27ccc917

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

* Credential add Jenkins netdevopsuser
* Branch */dev
* Repository url = http://198.18.134.48:3000/netdevopsuser/netdevops-jenkins-ansible.git
* Repository Browser = GOGs
* save and go back and review pipeline config

![Picture23](https://user-images.githubusercontent.com/11307137/55590698-a0733780-5701-11e9-88cb-5bc28e3013ea.png)
![Picture24](https://user-images.githubusercontent.com/11307137/55590734-baad1580-5701-11e9-85a5-afd6ab52d9f8.png)

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

<img width="1070" alt="Screenshot 2019-04-11 23 30 30" src="https://user-images.githubusercontent.com/11307137/56012380-e146ef80-5cb1-11e9-93a8-fcccea4f31ed.png">

After clicking "Create Bot", you'll be presented with the Bot Details. At the bottom of the details is the "Bot's Access Token". Click the button to "Copy" the token.  Make sure to note the Token and not mix up with BOT ID.

<img width="847" alt="Screenshot 2019-04-11 23 34 13" src="https://user-images.githubusercontent.com/11307137/56012485-57e3ed00-5cb2-11e9-9aaa-f798124691b5.png">

Paste token into a text file for safe keeping!

Now we need to add a new room to webex teams and add the BOT we just created:

![Screenshot 2019-04-11 14 57 18](https://user-images.githubusercontent.com/11307137/55988770-5989c280-5c6a-11e9-9211-f2ac4f364fcc.png)

Add your Bot that you created above to the room:

![Screenshot 2019-04-11 15 00 26](https://user-images.githubusercontent.com/11307137/55988861-95bd2300-5c6a-11e9-8273-071cdcb7bbc9.png)

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
[developer@centos netdevops-jenkins-ansible]$cd ansible-04-mission/
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
[developer@centos netdevops-jenkins-ansible]$cd ansible-04-mission/
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
CD back to netdevops-jenkins-ansible and edit Jenkins file to edit Notifications in pipeline.

```
[developer@centos ansible-04-mission]$cd ..
[developer@centos netdevops-jenkins-ansible]$ls
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

[developer@centos netdevops-jenkins-ansible]$git add Jenkinsfile
[developer@centos netdevops-jenkins-ansible]$git commit -m "notifications"
[dev e8552d9] notifications
 3 files changed, 5 insertions(+), 5 deletions(-)
[developer@centos netdevops-jenkins-ansible]$git push
Counting objects: 11, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (6/6), 714 bytes | 0 bytes/s, done.
Total 6 (delta 5), reused 0 (delta 0)
Username for 'http://198.18.134.48:3000': netdevopsuser
Password for 'http://netdevopsuser@198.18.134.48:3000':
To http://198.18.134.48:3000/netdevopsuser/netdevops-jenkins-ansible.git
   4380055..e8552d9  dev -> dev
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'http://198.18.134.48:3000/netdevopsuser/netdevops-jenkins-ansible.git'
hint: Updates were rejected because a pushed branch tip is behind its remote
hint: counterpart. Check out this branch and merge the remote changes

```

# GOGs and Jenkins Integration
At this point we can prepare the Gogs server to integrate with the Jenkins server. Click on settings.

<img width="617" alt="Screenshot 2019-04-12 14 49 04" src="https://user-images.githubusercontent.com/11307137/56059487-c9ab4d80-5d29-11e9-8a8c-b056eea3522c.png">


Click webhooks:

<img width="636" alt="Screenshot 2019-04-12 14 53 10" src="https://user-images.githubusercontent.com/11307137/56059748-c9fc1680-5d32-11e9-8952-c268f3faf176.png">

Add webhook + Gogs:


![Picture31](https://user-images.githubusercontent.com/11307137/55595701-20ee6400-5713-11e9-8685-80422eda138c.png)

Add webhook:
payload url = http://198.18.134.48:8080/gogs-webhook/?job=pipeline
secret = network


<img width="630" alt="Screenshot 2019-04-12 14 58 56" src="https://user-images.githubusercontent.com/11307137/56059994-8d7cea80-5d33-11e9-8248-61cc015b7310.png">


# Run Pipeline and review results
Make sure the pipeline runs successfully by reviewing the console output and Cisco Spark Bot notifications. One method to run the pipeline is to initiate the pipeline by making a change to code in our GOGs repository. The Jenkinfile will kickoff the pipeline when dedecting changes to files. We create a change event by adding a comment line to the Jenkinsfile and then commit our change to GIT to kickoff the pipeline. 


Make changes to Jenkinsfile.

```
[developer@centos netdevops-jenkins-ansible]$ vim Jenkinsfile

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

Check Webex Team Space "Spark". You will see three sets of notifications. The first set refers to the Dev code changes, while the second set refers to Merging changes to the Master Branch. The third verifies the production changes were completed. When you make a change to the Jenkinsfile it will send the Notifications out twice because it will run first from the Dev branch and then run again from the Master Branch. We need to remove the Jenkinsfile from the master branch to stop the redundant pipeline build.

<img width="457" alt="Screenshot 2019-04-05 00 23 02" src="https://user-images.githubusercontent.com/11307137/55603507-ffa06e80-5738-11e9-8603-4a2a477576a9.png">

## Fixing the Jenkinsfile issue to avoid the redundant build issue

```
[developer@centos netdevops-jenkins-ansible]$ git checkout master
git rm Jenkinsfile
touch .gitignore
echo 'Jenkinsfile' > .gitignore
git add .gitignore
git commit -m 'ignore Jenkinsfile in master branch'
git push --force
git checkout dev
```
Sometimes you have to run git push --force twice and then git checkout dev, if you receive the following error:

```
fatal: The remote end hung up unexpectedly
```

Any new changes or pipeline jobs ran from Jenkins should only run once from this point forward.

# Bonus Step: Install PyATS pllugin for Jenkins

Clone PyATS plugin from Github to your laptop

<img width="1196" alt="Screenshot 2019-04-10 22 32 49" src="https://user-images.githubusercontent.com/11307137/55929207-38c95a80-5be1-11e9-9186-3b04dae44773.png">

Open up Jenkins GUI and manage plugins:

<img width="1329" alt="Screenshot 2019-04-10 22 40 35" src="https://user-images.githubusercontent.com/11307137/55929328-b725fc80-5be1-11e9-9097-4f39e038b907.png">

Goto advanced to upload .hpi plugin file:

<img width="1074" alt="Screenshot 2019-04-10 22 31 31" src="https://user-images.githubusercontent.com/11307137/55929356-d91f7f00-5be1-11e9-8030-597af5487fdb.png">

Make sure the .hpi 4.0 plugin installs successfully in Jenkins:

<img width="676" alt="Screenshot 2019-04-10 22 33 43" src="https://user-images.githubusercontent.com/11307137/55929411-24399200-5be2-11e9-9a27-a931d55eee05.png">

# Bonus 2 (Optional) Using your local laptop ansible to connect to the GOGs:

```
On your laptop:
git clone https://github.com/andubiel/netdevops-jenkins-ansible.git
cd netdevops-jenkins-ansible
git config --local user.name "netdevopsuser"
git config --local user.email "netdevopsuser@netdevops.local"
git fetch
git checkout dev 
```




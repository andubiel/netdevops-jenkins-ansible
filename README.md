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













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

Change GOGs URL to 198.18 134.48 if not already displaying and then start GOGs.

![Picture5](https://user-images.githubusercontent.com/11307137/55587483-6ef66e00-56f9-11e9-9ece-5dd2d486ef59.png)








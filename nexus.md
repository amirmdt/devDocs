# How to Install

first u need to install wget package on your rocky linux

and java package too `sudo dnf install java` ( use `java -version` command to check if java is installed)
____________________________
____________________________
### install java using its `tar.gz` file
first, download the `tar.gz` file of java jdk in version you need `google: java jdk tar.gz download`
```
wget https://builds.openlogic.com/downloadJDK/openlogic-openjdk/17.0.14+7/openlogic-openjdk-17.0.14+7-linux-x64.tar.gz
```
unzip the `tar.gz` file:
```
tar -xvf openlogic-openjdk-17.0.14+7-linux-x64.tar.gz
```
rename the folder refer to openjdk to jdk-17:
```
mv openlogic-openjdk-17.0.14+7-linux-x64 jdk-17
```
Set Java path temporarily:
```
export JAVA_HOME=/opt/jdk-17
export PATH=$JAVA_HOME/bin:$PATH
```
then verify:
```
java --version
```
To make it permanent:
```
sudo vim /etc/profile.d/java.sh
```
paste:
```
export JAVA_HOME=/opt/jdk-17
export PATH=$JAVA_HOME/bin:$PATH
```
Then:
```
sudo chmod +x /etc/profile.d/java.sh
source /etc/profile.d/java.sh
```
now the java should be installed.
____________________________
____________________________

then follow the below commands :

`cd /opt` *to go to /opt directory*

*find the latest version of nexus tar.gz file (in its site) the copy the link address for download it and use command like below to get that tar.gz package*

`wget https://download.sonatype.com/nexus/3/nexus-3.79.1-04-linux-x86_64.tar.gz`

*then unzip the package you downloaded using command* `tar -xvf nexus-3.79.1-04-linux-x86_64.tar.gz`

*if you use command* `ls` *you'll see there are 2 directories* ***nexus-3.79.1-04*** *and* ***sonatype-work***

*change the name of first directory to nexus* `mv nexus-3.79.1-04 nexus`

*create a new user named* ***nexus*** `adduser nexus`

*change owner and group owner of 2 directories to nexus using commands below :*

`chown -R nexus:nexus /opt/nexus`

`chown -R nexus:nexus /opt/sonatype-work`

*create file* ***/opt/nexus/bin/nexus.rc*** *and write* ***`run_as_user="nexus"`*** *in it*

*you can edit file* ***/opt/nexus/bin/nexus.vmoptions***

## Run nexus as a Service

*create file* ***/etc/systemd/system/nexus.service*** *and write below configs in it*
```
[Unit]
Description=nexus service
After=network.target
[Service]
Type=forking
LimitNOFILE=65536
User=nexus
Group=nexus
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort
[Install]
WantedBy=multi-user.target
```
*then use followin commands to start the nexus service*\
`systemctl daemon-reload`\
`systemctl enable nexus.service`\
`systemctl start nexus.service`\
*and to ckeck if the service is started use command* `systemctl status nexus.service`

*to check log file of nexus:* `tail -f /opt/sonatype-work/nexus3/log/nexus.log`

*to check what port nexus is using* `netstat -tulnp | grep java`

## connect nexus web page

*use web browser to connect to url below*\
***`http://<nexus host ip>:<nexus port>`*** *for example mine is* ***`http://192.168.162.4:8081`***
____________________
____________________
### Problem: after i installed `nexus` and the service was running correctly and the java port was correct using `netstat -tulnp` i tried to connect nexus web page but i couldnt reach it. 
first i checked if Nexus is bound to the right interface?
Your netstat output shows:
```
tcp6       0      0 :::8081       :::*       LISTEN      2999/java
```
That means Nexus is listening on all IPv6 and IPv4 interfaces (:::8081 is the IPv6 equivalent of 0.0.0.0:8081), which is good.
Still, let’s double-check the nexus-default.properties config file:
```
cat /opt/nexus/etc/nexus-default.properties
```
Look for this line:
```
application-port=8081
application-host=0.0.0.0
```
If `application-host` is set to `127.0.0.1`, it means it's only accessible locally. You should change it to:
```
application-host=0.0.0.0
```
Then restart Nexus:
```
sudo systemctl restart nexus
```
but it wasn't my problem.
so i Checked Rocky Linux Firewall (firewalld):
Check the current rules:
```
sudo firewall-cmd --list-all
```
Allow port 8081:
```
sudo firewall-cmd --permanent --add-port=8081/tcp
sudo firewall-cmd --reload
```
after i allowed port 8081 in rocky linux firewall i could connect to the nexus web page.
____________________
____________________

*after you see the web page you should login with user admin which its password is in file* ***`/opt/sonatype-work/nexus3/admin.password`***

# Create a repository

*to create a repository in Web page : goto* ***Settings > Repositories > Create repository***

# hosted, group and proxy Repositories


# Upload a package

*to upload a package to repository you should use command below*\
`curl -u admin:password --upload-file /pathto/packageName http://nexsus_address:8081/repository/reponame/<optional_direcories>/packageName`\
*just be careful of Repodata Depth because if you upload your file in depth lower than the* ***Repodata Depth*** *it won't be uploaded*

# Download a package

*to download a package use command below*\
`curl -O http://nexus_address:8081/packageName`

_________________________
_________________________
### Tip: if you want to download the file using this command, you should enable the `Anonymous Access` in nexus web in settings. but if the `Anonymous Access` is not enabled you should use command below:
```
curl -O http://nexus_address:8081/packageName -u username:password
```
_________________________
_________________________

# How to Download rpm files from yum hosted Repository using **dnf** command without downloading it

# yum proxy Repository


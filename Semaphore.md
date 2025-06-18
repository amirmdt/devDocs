## Install

*first install a Debian machine (need internet access)*\
*then create a ***system*** user which has ***home directory*** using command below:*
```
sudo adduser --system --group --home /home/semaphore semaphore
```

*after that we need a database server.*
```
sudo apt update
sudo apt install mariadb-server
```
*to make sure it's installed :*
```
systemctl status mariadb
```
*to use mariadb:*
```
sudo mariadb
```
*and to exit from there:*
```
exit
```
*give password to ***root*** user of mysql (mariadb)*
```
sudo mysql_secure_installation
```
*answer to question below* ***n***
```
Swirch to unix-socket authentication [Y/n] n
```
*then if you didnt set password for root in the next question change the password*

*next questions and answers :*
```
Remove anonymous users? Y
Disallow root login remotely? Y
Remove test database and access to it? Y
Reload privilege tables now? Y
```
*then go back to mariadb*
```
sudo mariadb
```
and in it:
```
CREATE DATABASE semaphore_db;
SHOW DATABASES;
GRANT ALL PRIVILEGES ON semaphore_db.* TO semaphore_user@localhost IDENTIFIED BY 'semaphore_user_password';
FLUSH PRIVILEGES; #to make sure everything take effect.
exit
```

*then using a web browser go to link below:*\
`github.com/ansible-semaphore/semaphore/releases/latest`\
*and copy the link address of the release you need ( in my case `semaphore_2.13.14_linux_amd64.deb` ) which the link is like:*\
`https://github.com/semaphoreui/semaphore/releases/download/v2.13.14/semaphore_2.13.14_linux_amd64.rpm`

**then using command below download it on debian machine:*
```
wget https://github.com/semaphoreui/semaphore/releases/download/v2.13.14/semaphore_2.13.14_linux_amd64.rpm
```
*then install it:*
```
sudo apt install semaphore_2.13.14_linux_amd64.deb
```
*run command below to create the config file:*
```
semaphore setup
```
answer questions:
```
What database to use: > 1 - MySQL
db Hostname > Enter empty for default
db User > semaphore_user
db Password > semaphore_user_password
db Name > semaphore_db
Playbook path > Enter empty for default
Public URL > empty
Enable email alerts > empty to no
all other quesions empty to default
```
*then it will ask you to set username and email and name and password for next semaphore login (semaphore web login)*

then if we use `ls -l` command we'll see `config.json` file in the path we are. it contains answers we gave to the questions. ***if you want change something you can do that here.***

*now we change the owner and group of this config file to ***semaphore*** using command below:*
```
sudo chown semaphore:semaphore config.json
```
*then:*
```
sudo mkdir /etc/semaphore
sudo chown semaphore:semaphore /etc/semaphore
sudo mv config.json /etc/semaphore/
sudo apt install ansible
```
*to test semaphore and see if it's working :(we have some configs to add)*
```
semaphore server --config /etc/semaphore/config.json
```
to access semaphore in web explorer :\
`localhost:3000` or `<semaphore_server_IP>:3000`\
now use the `username` and `password` you set for semaphore to sign in.

### systemd service

the next thing we're going to do is to create a ***systemd service*** so help us automatically start semaphore, every time we start the service. to do that :
```
sudo nano /etc/systemd/system/semaphore.service
```
and put below texts in that file:
```
[Unit]
Description=Ansible Semaphore
Documentation=https://docs.ansible-semaphore.com/
Wants=network-online.target
After=network-online.target
ConditionPathExists=/usr/bin/semaphore
ConditionPathExists=/etc/semaphore/config.json

[Service]
ExecStart=/usr/bin/semaphore server --config /etc/semaphore/config.json
ExecReload=/bin/kill -HUP $MAINPID
Restart=always
RestartSec=10s
User=semaphore
Group=semaphore

[Install]
WantedBy=multi-user.target
```
next, we should reload the `systemd` :
```
sudo systemctl daemon-reload
```
this command will refresh the systemd to let it know we created a new service file\
then we enable and start the semaphore service :
```
sudo systemctl enable semaphore.service
sudo systemctl start semaphore.service
```
to check the status of service:
```
systemctl status semaphore.service
```
______________________________
### work with semaphore
in web page : `New Project > Project Name > CREATE`

this project will keep everything related to your `project` like : repository, ssh keys, inventory files and etc

then using web browser go to `github.com` and sign in. then we create a `repository` and it then we are going to create a `playbook` and `semaphore` is going to run the `playbook` for us.

then copy the https address of github repository just created.

then in semaphore web page :\
* `NEW REPOSITORY` then paste the https address copied as `URL or path` for `Branch` we chose `main` and for `Access Key` we chose `None` but if the github repository is `private` we should change it. > CREATE
* go to `Environment` : `NEW ENVIRONMENT` name it (for example ***Production***) in we added in `Extra variables` part :
```
{
  "var_environment": "production"
}
```
* then in `Key Store` part: `NEW KEY` > name the key (e.g. `my key`) > Type : `SSH Key` > Username > `username of the server`(which we set in semaphore setup to web login) > for the `Private Key` part we should generate a ssh key in server terminal using command `ssh-keygen` then copy the `private key file` and paste it in semaphore `Private KEY` part.
* then in `Inventory`: `NEW INVENTORY` > name the inventory > User Credentials : the key created in previous part > Type : Static > and fill in blank space with your yaml inventory > CREATE
* then in `Task Templates` : `NEW TEMPLATE` and choose `Inventory`, `Repository` and `Variable Group` what we created just a few moments ago then > CREATE

### Next step

as the next step, in target hosts we should add their users in sudoes to need no password to do sudo jobs.
for that we can do each of these 2 ways :
1. create a file named like the target host username ( for example in mine its amirmohamad) in the path `/etc/sudoers.d/` and in this wrtie : `amirmohamad ALL=(ALL) NOPASSWD: ALL`
2. edit file `/etc/sudoers` (this file is not writeable, so you should chmod it to 660 after you did your edit chamod back it to 440) and add `amirmohamad ALL=(ALL) NOPASSWD: ALL` to this file

after that you should add your semaphore_server ( the debian server which we installed semaphore on it ) pub ssh key which created before to target hosts. using one of these ways:

1. using command : `ssh-copy-id -i ~/.ssh/id_rsa.pub host1@192.168.12.4`
2. add contents of `~/.ssh/id_rsa.pub` to `~/.ssh/authorized_keys` file in target hosts

__________________
then in github : in the repository created for semaphore, ***create a new file*** and fill it with :
```
---
- hosts: vservers (my_servers)
  become: true
  tasks:
    - name: isntalle Apache
      ansible.builtin.package:
        name: apache2
```
and give the file a name (e.g. `site.yml`) and Commit changes.

now before running the `task` copy and paste the ip address of the target host in the browser. nothing will happen.

now go back to semaphore web, go to `Task Templates` and run the Task Created (Click RUN and RUN again)

after running the Task if you go to ip address of target host using browser, it will show you home page of Apache2.



________________________________________


```
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        server_name semaphore.amirmdt.com;

    location / {
      proxy_cache_bypass $http_upgrade;
      proxy_http_version 1.1;
      proxy_pass http://localhost:3000;
      proxy_set_header Connection 'upgrade';
      proxy_set_header Host $host;
      proxy_set_headr Upgrade $http_upgrade;
    }
}
```

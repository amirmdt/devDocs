## PasswordLess SSH :

1. *in client use command below :*\
`ssh-keygen -t rsa`\
*it will create 2 key files one pub and one private*\
*( while creating files you can choose where to create files and you can write a password as passphrase which will be asked while using this keys for ssh [you can enter to choose default file position and no passphrase] )*

2. *copy the public file to the `.ssh` directory in destination computer and cat and append ( >> ) its contents to `authorized_keys` file in .ssh directory in destination computer ( if there is no `authorized_keys` file in destination , create one )*

3. *`chmod 700 ~/.ssh`*\
 *`chmod 600 ~/.ssh/authorized_keys`*


4. *now you can connect ssh from client to destination with no password ( if you choose passphrase you should enter while using ssh )*


## Proxy Jump SSH :

*just with command below you can connect to destination server by passing through one or more jumper servers :*

`ssh -J user1@jumper1 user2@jumper2 … userdest@destination` 

`ssh -J <jump server> <remote server>`

`ssh -J <jump server1>,<jump server2>,<jump server3> <remote server>`


## Simple SSH :

*in client server in `~/.ssh/` directory in `config` file ( if there is no config file create `~/.ssh/config` file ) you can write like below to use easy SSH :*
```
Host remoteserver
  HostName 192.168.200.200
  User dev
  IdentityFile ~/.ssh/<your_key>
  Port 2048
```

*for example I did it in mine like below :*
```
Host remoteserver
	HostName 192.168.56.102
	user amir
	IdentityFile ~/.ssh/id_rsa
	Port 22
Host windows11
	HostName 192.168.56.1
	user Amir
	Port 22
```
*and to ssh to `192.168.56.102` I only need to use :*\
`ssh remoteserver`\
*command  and to proxy jump from one to another I do like below :*\
`ssh -J windows11 remoteserver`




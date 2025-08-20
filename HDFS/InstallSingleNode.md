## 🔹 Steps to Install HDFS (Hadoop) on Rocky Linux (Single Node)

We’ll install Hadoop, since HDFS is part of it.

### 1. Prerequisites

#### Make sure your system is updated:
```
sudo dnf update -y
```
### 2. Create a Hadoop User
It’s best not to run Hadoop as root:
```
sudo adduser hadoop
sudo passwd hadoop
```
Give it sudo access:
```
sudo usermod -aG wheel hadoop
```
Switch to that user:
```
su - hadoop
```
### 3. Install Java (Hadoop needs Java):
download the OpenJDK11 binary:
```
wget https://github.com/AdoptOpenJDK/openjdk11-binaries/releases/download/jdk-11.0.9.1%2B1/OpenJDK11U-jdk_x64_linux_hotspot_11.0.9.1_1.tar.gz
```
go to `/usr/local` and extract it:
```
cd /usr/local
sudo tar -xvzf /tmp/OpenJDK11U-jdk_x64_linux_hotspot_11.0.9.1_1.tar.gz
```
This will create a folder like:
```
/usr/local/jdk-11.0.9.1+1
```
Set `JAVA_HOME` and PATH:\
Add to your `~/.bashrc` (for the `hadoop` user):
```
export JAVA_HOME=/usr/local/jdk-11.0.9.1+1
export PATH=$JAVA_HOME/bin:$PATH
```
Reload:
```
source ~/.bashrc
```
Verify:
```
java -version
```
you should see:
```
openjdk version "11.0.9.1" 2020-11-04
OpenJDK Runtime Environment AdoptOpenJDK (build 11.0.9.1+1)
OpenJDK 64-Bit Server VM AdoptOpenJDK (build 11.0.9.1+1, mixed mode)
```

### 4. Download Hadoop
Go to [Apache Hadoop releases](https://hadoop.apache.org/releases.html), but let’s grab latest stable 3.x with wget:
```
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.4.1/hadoop-3.4.1.tar.gz
```
Extract it:
```
tar -xvzf hadoop-3.4.1.tar.gz
mv hadoop-3.4.1 hadoop
```
### 5. Set Environment Variables
Edit your `.bashrc`:
```
vim ~/.bashrc
```
Add:
```
export HADOOP_HOME=/home/hadoop/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export JAVA_HOME=/usr/local/jdk-11.0.9.1+1
export PATH=$JAVA_HOME/bin:$PATH
```
Reload:
```
source ~/.bashrc
```
### 6. Configure Hadoop (Single Node)
Go to config directory:
```
cd $HADOOP_CONF_DIR
```
Edit `core-site.xml`:
```
<configuration>
   <property>
       <name>fs.defaultFS</name>
       <value>hdfs://localhost:9000</value>
   </property>
</configuration>
```
Edit `hdfs-site.xml`:
```
<configuration>
   <property>
       <name>dfs.replication</name>
       <value>1</value>
   </property>
   <property>
       <name>dfs.namenode.name.dir</name>
       <value>file:/home/hadoop/hadoopdata/namenode</value>
   </property>
   <property>
       <name>dfs.datanode.data.dir</name>
       <value>file:/home/hadoop/hadoopdata/datanode</value>
   </property>
</configuration>
```
Create the folders:
```
mkdir -p ~/hadoopdata/namenode
mkdir -p ~/hadoopdata/datanode
```
Edit `hadoop-env.sh`\
Find the line for Java and set:
```
export JAVA_HOME=/usr/local/jdk-11.0.9.1+1
```
### 7. Format the HDFS Namenode
```
hdfs namenode -format
```
### 8. Start HDFS
```
start-dfs.sh
```
Tip: when i ran this command i got error below:
```
Starting namenodes on [localhost] localhost: Warning: Permanently added 'localhost' (ED25519) to the list of known hosts. localhost: hadoop@localhost: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password). Starting datanodes localhost: hadoop@localhost: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password). Starting secondary namenodes [localhost.localdomain] localhost.localdomain: Warning: Permanently added 'localhost.localdomain' (ED25519) to the list of known hosts. localhost.localdomain: hadoop@localhost.localdomain: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
```
What’s happening:\
When you run `start-dfs.sh`, Hadoop tries to SSH into `localhost` as the hadoop user in order to start the NameNode, DataNode, and SecondaryNameNode daemons.

Right now, your `hadoop` user cannot SSH into localhost without a password, so Hadoop fails with:
```
Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password)
```
#### 🔹 Fix: Enable passwordless SSH for the hadoop user
Step 1: Switch to the hadoop user
```
su - hadoop
```
Step 2: Generate SSH keys
```
ssh-keygen -t rsa
```
Press Enter for all prompts → it will create keys in `~/.ssh/id_rsa`.

Step 3: Copy the key to localhost
```
ssh-copy-id hadoop@localhost
```
If it asks for a password, enter the `hadoop` user’s password once.\
This will add the key to `~/.ssh/authorized_keys`.
Step 4: Test\
Still as hadoop user:
```
ssh localhost
```
It should log in without asking for a password.\
Type exit to return
#### 🔹 Retry Hadoop
Now start HDFS again:
```
start-dfs.sh
```
Check if processes are running:
```
jps
```
You should see:
- NameNode
- DataNode
- SecondaryNameNode

### 9. Test HDFS
Create a test directory:
```
hdfs dfs -mkdir /test
hdfs dfs -ls /
```
Put a file:
```
echo "Hello HDFS" > hello.txt
hdfs dfs -put hello.txt /test/
hdfs dfs -ls /test
```
Read it back:
```
hdfs dfs -cat /test/hello.txt
```
✅ At this point, you have a single-node HDFS running on Rocky Linux.















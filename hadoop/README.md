# Hadoop #

## Single Node ##

```bash
# Configure SSH: passphraseless ssh key
sudo apt-get install -y ssh
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
echo 'StrictHostKeyChecking no' >> ~/.ssh/config

# Configure PDSH: Use ssh for rcmd instead of rsh
sudo apt-get install -y pdsh
sudo bash -c "echo 'ssh' > /etc/pdsh/rcmd_default"

# Download Hadoop
HADOOP_VERSION=3.2.1
wget -O hadoop.tar.gz https://archive.apache.org/dist/hadoop/common/hadoop-${HADOOP_VERSION}/hadoop-${HADOOP_VERSION}.tar.gz
mkdir hadoop
tar xf hadoop.tar.gz --strip-components=1 -C hadoop
cd hadoop

# configure Java: Set JAVA_HOME
# See [Hadoop Java Versions](https://cwiki.apache.org/confluence/display/HADOOP/Hadoop+Java+Versions)
sudo apt-get install -y openjdk-8-jdk
sed -i 's@^# export JAVA_HOME=.*@export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64@' etc/hadoop/hadoop-env.sh

# Configure Hadoop
cat <<EOF | sed -i '/<configuration>/r /dev/stdin' etc/hadoop/core-site.xml
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
EOF
cat <<EOF | sed -i '/<configuration>/r /dev/stdin' etc/hadoop/hdfs-site.xml 
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
EOF

# Start Hadoop in pseudo-distributed mode locally
./bin/hdfs namenode -format     # Format NameNode only at first time
./sbin/start-dfs.sh             # Start HDFS (NameNode and DataNode)
firefox http://localhost:9870/  # View NameNode status

# Make required HDFS directories
./bin/hdfs dfs -mkdir /user
./bin/hdfs dfs -mkdir /user/<username>

# Run an example job
./bin/hdfs dfs -mkdir input                 # Create a directory
./bin/hdfs dfs -put etc/hadoop/*.xml input  # Upload files to the directory
./bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar grep input output 'dfs[a-z.]+'
./bin/hdfs dfs -cat output/*                # View results

# Stop Hadoop
./sbin/stop-dfs.sh

# Configure YARN
cat <<EOF | sed -i '/<configuration>/r /dev/stdin' etc/hadoop/mapred-site.xml
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>\$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:\$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
EOF
cat <<EOF | sed -i '/<configuration>/r /dev/stdin' etc/hadoop/yarn-site.xml
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
EOF

# Start Hadoop on YARN in pseudo-distributed mode
./sbin/start-dfs.sh             # Start HDFS (NameNode and DataNode)
./sbin/start-yarn.sh            # Start YARN (ResourceManager and NodeManager)
firefox http://localhost:9870/  # View NameNode status
firefox http://localhost:8088/  # View ResourceManager status

# Run an example job as the above

# Stop Hadoop
./sbin/stop-dfs.sh
./sbin/stop-yarn.sh
```

## Multiple Nodes ##

Take Hadoop 2.7.4 as an example, since this version is by default
included in PySpark 3.0.0.  See
* [Hadoop: Setting up a Single Node Cluster](https://hadoop.apache.org/docs/r2.7.4/hadoop-project-dist/hadoop-common/SingleCluster.html)
* [Hadoop Cluster Setup](https://hadoop.apache.org/docs/r2.7.4/hadoop-project-dist/hadoop-common/ClusterSetup.html)

Hadoop is configured via several XML files:
* `hadoop/etc/hadoop/core-site.xml`
* `hadoop/etc/hadoop/hdfs-site.xml`
* `hadoop/etc/hadoop/yarn-site.xml`
* `hadoop/etc/hadoop/mapred-site.xml`

These configurations' default values are defined in the corresponding
XML files:
* `core-default.xml`
* `hdfs-default.xml`
* `yarn-default.xml`
* `mapred-default.xml`


```bash
# Setup environment variables.
# Let Hadoop script know where all the stuffs are located if the paths
# are different on different nodes.
# Seems that it will try to figure out these paths if not set.
# However, once set, it won't try to figure out these paths by itself.
# So 
cd ~/
cat << EOF >> ~/.bashrc
export HADOOP_PREFIX=~/hadoop  # Hadoop version 2.7.4
export HADOOP_HOME=~/hadoop    # Latest version of Hadoop uses HADOOP_HOME instead of HADOOP_PREFIX
export HADOOP_CONF_DIR=\${HADOOP_PREFIX}/etc/hadoop
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
EOF
source ~/.bashrc

# Setup phraseless ssh.
# Make sure all nodes has the same `~/.ssh/authorized_keys` file.
# Check on Azure VMs:
#     $ ssh localhost
#     $ ssh 172.16.4.4
#     $ ssh 172.16.4.5
#     $ ssh 172.16.4.6
sudo apt-get update
sudo apt-get install -y ssh rsync
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
echo 'StrictHostKeyChecking no' >> ~/.ssh/config

# Download Hadoop of specific version
HADOOP_VERSION=2.7.4
wget -O hadoop.tar.gz https://archive.apache.org/dist/hadoop/common/hadoop-${HADOOP_VERSION}/hadoop-${HADOOP_VERSION}.tar.gz
mkdir hadoop
tar xf hadoop.tar.gz --strip-components=1 -C hadoop
rm hadoop.tar.gz

# Setup Java.  Check:
#     $ ./hadoop/bin/hadoop
sudo apt-get install -y openjdk-8-jdk
# Run the command below if JAVA_HOME is not set previously
sed -i 's@^export JAVA_HOME=.*@export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64@' hadoop/etc/hadoop/hadoop-env.sh

DFS_TMP_DIR=/home/simon/dfs
cat <<EOF | sed -i '/<configuration>/r /dev/stdin' hadoop/etc/hadoop/hdfs-site.xml
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>${DFS_TMP_DIR}/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>${DFS_TMP_DIR}/data</value>
    </property>
EOF

# Configure Hadoop
NAMENODE_IP=172.16.4.4
cat <<EOF | sed -i '/<configuration>/r /dev/stdin' hadoop/etc/hadoop/core-site.xml
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://${NAMENODE_IP}:9000</value>
    </property>
EOF

RESOURCEMANAGER_IP=172.16.4.4
cat <<EOF | sed -i '/<configuration>/r /dev/stdin' hadoop/etc/hadoop/yarn-site.xml
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>${RESOURCEMANAGER_IP}</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
EOF

JOBHISTORY_IP=172.16.4.4
cp hadoop/etc/hadoop/mapred-site.xml.template hadoop/etc/hadoop/mapred-site.xml
cat <<EOF | sed -i '/<configuration>/r /dev/stdin' hadoop/etc/hadoop/mapred-site.xml
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
      <name>mapreduce.jobhistory.address</name>
      <value>${JOBHISTORY_IP}:10020</value>
    </property>
    <property>
      <name>mapreduce.jobhistory.webapp.address</name>
      <value>${JOBHISTORY_IP}:19888</value>
    </property>
EOF

# Specify worker nodes.
# Only needs to be set on the namenode.
# NOTE: The newest version uses etc/hadoop/workers instead of etc/hadoop/slaves
cat << EOF > hadoop/etc/hadoop/slaves
172.16.4.4
172.16.4.5
172.16.4.6
EOF

# Start Hadoop daemons
./hadoop/bin/hdfs namenode -format     # Format NameNode only at first time
./hadoop/sbin/start-dfs.sh
./hadoop/sbin/start-yarn.sh
./hadoop/sbin/mr-jobhistory-daemon.sh start historyserver

# Check all daemons are running.
# Make sure NameNode, ResourceManager, DataNode, NodeManager are running on
# corresponding nodes.
# If DataNode is not running, stop all daemons, remove DFS_TMP_DIR, format namenode,
# then start all daemons again.
jps

# Run an example to check if Hadoop cluster is ready
./hadoop/bin/hdfs dfs -mkdir -p /user/simon
./hadoop/bin/hdfs dfs -put hadoop/README.txt .
./hadoop/bin/hdfs dfs -ls
./hadoop/bin/yarn jar ~/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.4.jar wordcount "./*" output
./hadoop/bin/hdfs dfs -cat output/part-r-00000

# Web Interfaces
firefox http://${NAMENODE_IP}:50070 &
firefox http://${RESOURCEMANAGER_IP}:8088 &
firefox http://${JOBHISTORY_IP}:19888 &

# Stop Hadoop daemons
./hadoop/sbin/stop-dfs.sh
./hadoop/sbin/stop-yarn.sh
./hadoop/sbin/mr-jobhistory-daemon.sh stop historyserver
```


## Reference ##

* [Hadoop Documentation](https://hadoop.apache.org/docs/stable/)
    + [Hadoop: Setting up a Single Node Cluster (Latest Version)](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html)
    + [Hadoop: Setting up a Single Node Cluster (Version 2.7.4)](https://hadoop.apache.org/docs/r2.7.4/hadoop-project-dist/hadoop-common/SingleCluster.html)
    + [Hadoop Cluster Setup (Latest Version)](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/ClusterSetup.html)
    + [Hadoop Cluster Setup (Version 2.7.4)](https://hadoop.apache.org/docs/r2.7.4/hadoop-project-dist/hadoop-common/ClusterSetup.html)
* [Hadoop Dev Dockerfile](https://github.com/apache/hadoop/tree/trunk/dev-support/docker)
* Other Hadoop Cluster Setup Tutorials
    + [How To Install Hadoop On Ubuntu 18.04 Or 20.04](https://phoenixnap.com/kb/install-hadoop-ubuntu)
    + [Apache Hadoop 3.x installation on Ubuntu (multi node cluster)](https://sparkbyexamples.com/hadoop/apache-hadoop-installation/)
        - [Yarn setup on Hadoop 3.1](https://sparkbyexamples.com/hadoop/yarn-setup-and-run-map-reduce-program/)
    + [How to Install and Set Up a 3-Node Hadoop Cluster](https://www.linode.com/docs/databases/hadoop/how-to-install-and-set-up-hadoop-cluster/)

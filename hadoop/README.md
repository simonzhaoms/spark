# Hadoop #

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
wget -O hadoop.tar.gz https://downloads.apache.org/hadoop/common/hadoop-3.2.1/hadoop-3.2.1.tar.gz
mkdir hadoop
tar xf hadoop.tar.gz --strip-components=1 -C hadoop
cd hadoop

# configure Java: Set JAVA_HOME
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
./bin/hdfs namenode -format     # Format NameNode
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
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
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

## Reference ##

* [Hadoop Documentation](https://hadoop.apache.org/docs/stable/)
    + [Getting Started -- Hadoop: Setting up a Single Node Cluster](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html)
    + [Hadoop Cluster Setup](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/ClusterSetup.html)

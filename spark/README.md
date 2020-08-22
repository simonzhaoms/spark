# Spark #

* [Installation](#installation)
* [Quickstart](#quickstart)
* [Run Spark Interactively](#run-spark-interactively)
* [Run Spark on Cluster](#run-spark-on-cluster)
    + [Run Spark Application in Batch Mode](#run-spark-application-in-batch-mode)
    + [Setup Spark's Own Standalone Cluster](#setup-sparks-own-standalone-cluster)
* [Spark Configuration](#spark-configuration)
* [Reference](#reference)


## Installation ##

1. Install prerequisites
    1. Java
        * Spark 2.4.x only support Java 8.  See [Spark Release
          2.4.1](https://spark.apache.org/releases/spark-release-2-4-1.html).
        * Spark 3.x support Java 11 but does not support Scala 2.11
          anymore.  See [Spark Release
          3.0.0](https://spark.apache.org/releases/spark-release-3-0-0.html).

       ```bash
       sudo apt-get update
       sudo apt-get install -y openjdk-8-jdk
       
       # Optional
       sudo update-alternatives --config java
       ```

    1. Python
        * From 3.0.0 on, PySpark support Python 3.8.  PySpark of lower
          versions does not.  See [PySpark does not work with Python
          3.8.0](https://issues.apache.org/jira/browse/SPARK-29536)
          and the 'Programming Language' section on
          [PyPI](https://pypi.org/project/pyspark/).
          
       ```bash
       sudo apt-get update
       sudo apt-get install -y python3
       
       # By default, Spark will find 'python' in the PATH
       ln -s /usr/bin/python3 ~/.local/bin/python
       ```

1. Install Spark
    * It can be downloaded and installed as binaries:
        + From [Apache Spark
          Downloads](https://spark.apache.org/downloads.html)
            - Choose a Spark release: `2.4.5 (Feb 05 2020)`
            - Choose a package type: `Pre-built for Apache Hadoop 2.7`
            - Download Spark: `spark-2.4.5-bin-hadoop2.7.tgz`
        + Older release can be found at [Apache Spark
          Archive](https://archive.apache.org/dist/spark/)

          ```bash
          SPARK_URL='http://archive.apache.org/dist/spark/spark-2.4.5/spark-2.4.5-bin-hadoop2.7.tgz'
          SPARK_FILE="${SPARK_URL##*/}"
          SPARK_DIR='spark'
          wget "${SPARK_URL}"
          mkdir "${SPARK_DIR}"
          tar xf "${SPARK_FILE}" --strip-components=1 -C "${SPARK_DIR}"
          ```

    * Or installed from PyPI as
      [PySpark](https://pypi.org/project/pyspark/):

      ```bash
      sudo apt install -y python3-pip
      pip3 install pyspark
      ```

    * See [PySpark Notes](pyspark.md) for more about the differences
      between PySpark and Spark.


## Quickstart ##

```bash
cat << EOF > test.py
from pyspark.sql import SparkSession

file = './spark/README.md'
spark = SparkSession.builder.appName('Test').getOrCreate()
text = spark.read.text(file).cache()
print(text.count())
spark.stop()
EOF

python test.py
```

Output:

```
20/08/17 08:41:14 WARN Utils: Your hostname, pdbionic resolves to a loopback address: 127.0.1.1; using 10.211.55.62 instead (on interface enp0s5)
20/08/17 08:41:14 WARN Utils: Set SPARK_LOCAL_IP if you need to bind to another address
20/08/17 08:41:14 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
104
```


## Run Spark Interactively ##

When Spark is run interactively, an instance of `SparkContext` and
`SparkSession` are automatically created by Spark.  They are called
`sc` and `spark`, respectively.

* Spark shell in Scala
    + Run without a cluster

      ```bash
      bash "${SPARK_DIR}"/bin/spark-shell
      ```

    + Or run on a cluster

      ```bash
      bash "${SPARK_DIR}"/bin/spark-shell --master <cluster-master-url>
      ```

  ```scala
  scala> sc
  res0: org.apache.spark.SparkContext = org.apache.spark.SparkContext@594965c9

  scala> spark
  res1: org.apache.spark.sql.SparkSession = org.apache.spark.sql.SparkSession@35599228

  scala> val x = spark.read.textFile("README.md")
  x: org.apache.spark.sql.Dataset[String] = [value: string]
  
  scala> x.count()
  res0: Long = 104
  
  scala> x.first()
  res2: String = # Apache Spark
  
  scala> val y = x.filter(line => line.contains("Spark"))
  y: org.apache.spark.sql.Dataset[String] = [value: string]
  ```

* Spark shell in Python
    + If Spark is installed as binaries, then
        * Run without a cluster

          ```bash
          bash "${SPARK_DIR}"/bin/pyspark
          ```

        * Or run on a cluster

          ```bash
          bash "${SPARK_DIR}"/bin/pyspark --master <cluster-master-url>
          ```

    + If Spark is installed as PySpark from PyPI, then
        * Run without a cluster

          ```bash
          pyspark
          ```

        * Or run on a cluster

          ```bash
          pyspark --master <cluster-master-url>
          ```

  ```pycon
  >>> sc
  <SparkContext master=local[*] appName=PySparkShell>
  >>> spark
  <pyspark.sql.session.SparkSession object at 0x7f90d4a6a320>
  >>> x = spark.read.text('README.md')
  >>> x
  DataFrame[value: string]
  >>> x.count()
  104
  >>> x.first()
  Row(value='# Apache Spark')
  >>> y = x.filter(x.value.contains('spark'))
  >>> y
  DataFrame[value: string]
  ```


## Run Spark on Cluster ##

See:
* [Cluster Mode Overview](https://spark.apache.org/docs/latest/cluster-overview.html)
* [Submitting Applications](https://spark.apache.org/docs/latest/submitting-applications.html)
* [Spark Standalone Mode](https://spark.apache.org/docs/latest/spark-standalone.html)

![cluster-overview](figure/cluster-overview.png)

Spark applications run as independent sets of processes on a cluster,
coordinated by the `SparkContext` object in their driver program and
executed by the executors run on worker nodes.  A driver program can
be launched inside (cluster deploy mode) or outside (client deploy
mode) of the cluster.  Supported cluster types are Spark's own
standalone cluster manager, Mesos, YARN and Kubernetes.


### Run Spark Application in Batch Mode ###

An Spark application can be
[submitted](https://spark.apache.org/docs/latest/submitting-applications.html)
in a batch mode by the `"${SPARK_DIR}"/bin/spark-submit` script, then
`SparkContext` in the driver program first connects [cluster
manager](https://spark.apache.org/docs/latest/cluster-overview.html#cluster-manager-types)
(such as Spark's own standalone cluster manager, Mesos, YARN, or
Kubernetes) to acquires executors on nodes in the cluster, then sends
application code and schedule tasks to the executors to run:

```bash
bash "${SPARK_DIR}"/bin/spark-submit \
    --class <main-class> \
    --master <cluster-master-url> \
    --deploy-mode <client/cluster> \
    --conf <key>=<value> \
    --packages <maven-coordinates> \
    <xxx.jar/xxx.py> [application-arguments]
```

where
* `<cluster-master-url>` can be
    + `local`: Run Spark locally with one worker thread
    + `local[K]`: Run Spark locally with `K` worker threads.
    + `spark://HOST:PORT`: Connect to the given Spark standalone
      cluster.
    + `yarn`: Connect to a YARN cluster.  Its location will be
      obtained from env variables `HADOOP_CONF_DIR` or
      `YARN_CONF_DIR`.
    + `k8s://HOST:PORT`: Connect to a Kubernetes cluster.
    + `mesos://HOST:PORT`: Connect to the given Mesos cluser.
* `<maven-coordinates>`: A comma-delimited list of Maven coordinates.


If the Spark application is written in Python and PySpark is
pip-installed, then it can be also run with the Python interpreter.
**NOTE** The cluster needs to be configured properly in the Python
code (the `xxx.py` file in the example below).

```bash
python xxx.py
```

### Run Spark on Hadoop Yarn cluster ###

First, setup a Hadoop Yarn cluster.  See
[Hadoop](../hadoop/README.md).  Then

```bash

```


### Setup Spark's Own Standalone Cluster ###

Besides running Spark locally with multiple threads, we can also run
Spark on a [Spark standalone
cluster](https://spark.apache.org/docs/latest/spark-standalone.html).
To setup a Spark standalone cluster:
* Manually one by one
  1. Start a master:
  
     ```bash
     bash "${SPARK_DIR}"/sbin/start-master.sh
     ```
     
     It will output a `<master-spark-url>` like `spark://HOST:PORT`.
  
  1. Start workers and connect them to the master above:
  
     ```bash
     bash "${SPARK_DIR}"/sbin/start-slave.sh <master-spark-url>
     ```

* Or use cluster launch scripts

  1. Create a file called `conf/slaves` to spesify worker hostname.
     And optionally the configuration file called `conf/spark-env.sh`.
     **NOTE**: The master machine should be able SSH to each of the
     workers.
  
     ```bash
     # Run the command below on the master instead of local machine
     bash "${SPARK_DIR}"/sbin/start-all.sh
     ```

## Spark Configuration ##

[Spark
properties](https://spark.apache.org/docs/latest/configuration.html#spark-properties)
can be set via the options below with precedence from highest to
lowest:
* `SparkConf` programmatically
* flags passed to `spark-submit`/`spark-shell`
* or options in the `${SPARK_HOME}/conf/spark-defaults.conf` file

Here `${SPARK_HOME}` is the same as `${SPARK_DIR}`.  If using PySpark,
`${SPARK_HOME}` is where it is installed.

Some properties may not be affected programmatically via `SparkConf`
in runtime, since they are related to deploy, such as
`spark.driver.memory`.

Besides using the default system Java and Python, Spark can also use
specified ones via [environment
variables](https://spark.apache.org/docs/latest/configuration.html#environment-variables)
set in the `${SPARK_HOME}/conf/spark-env.sh` file, such as `JAVA_HOME`
and `PYSPARK_PYTHON`, `HADOOP_CONF_DIR`.


See also [Allow specifying `HADOOP_CONF_DIR` as spark
property](https://github.com/apache/spark/pull/22289) for why not have
`HADOOP_CONF_DIR` as a property.


## Reference ##

* [Apache Spark](https://spark.apache.org/)
    + [Spark News](https://spark.apache.org/news/)
        + [Release Notes](https://spark.apache.org/releases/):
          Changes, Java compatibility, etc.
    + [Documentation](http://spark.apache.org/documentation.html)
        - [Latest Stable Version](https://spark.apache.org/docs/latest/index.html)
    + [Quick Start](https://spark.apache.org/docs/latest/quick-start.html)
    + [FAQ](https://spark.apache.org/faq.html)
    + [Building Spark](http://spark.apache.org/docs/latest/building-spark.html)
    + [Contributing to Spark](https://spark.apache.org/contributing.html)
    + [Useful Developer Tools](https://spark.apache.org/developer-tools.html)
* Testing
    * [Testing PySpark Code](https://mungingdata.com/pyspark/testing-pytest-chispa/)
* Spark with Hadoop Yarn
    * [Spark step-by-step setup on Hadoop Yarn cluster](https://sparkbyexamples.com/hadoop/spark-setup-on-yarn/)

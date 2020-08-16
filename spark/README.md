# Spark #

* [Installation](#installation)
* [Run Spark Interactively](#run-spark-interactively)
* [Run Spark on Cluster](#run-spark-on-cluster)
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


## Run Spark Interactively ##

* Spark shell in Scala

  ```bash
  cd "${SPARK_DIR}"
  "${SPARK_DIR}"/bin/spark-shell
  ```
  
  ```scala
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

      ```bash
      sudo ln -s /usr/bin/python3 ~/.local/bin/python  # Optional
      cd "${SPARK_DIR}"
      "${SPARK_DIR}"/bin/pyspark
      ```

    + If Spark is installed as PySpark from PyPI, then

      ```bash
      pyspark
      ```

  ```pycon
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
mode) of the cluster.

An Spark application can be
[submitted](https://spark.apache.org/docs/latest/submitting-applications.html)
by the `"${SPARK_DIR}"/bin/spark-submit` script, then `SparkContext`
in the driver program first connects [cluster
manager](https://spark.apache.org/docs/latest/cluster-overview.html#cluster-manager-types)
(such as Spark's own standalone cluster manager, Mesos, YARN,
Kubernetes) to acquires executors on nodes in the cluster, then sends
application code and schedule tasks to the executors to run:

```bash
"${SPARK_DIR}"/bin/spark-submit \
    --class <main-class> \
    --master <cluster-master-url> \
    --deploy-mode <client/cluster> \
    --conf <key>=<value> \
    --packages <maven-coordinates> \
    <application-jar/py> [application-arguments]
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

To run an interactive Spark shell against the cluster:

```bash
"${SPARK_DIR}"/bin/spark-shell --master <cluster-master-url>

# Or
"${SPARK_DIR}"/bin/pyspark --master <cluster-master-url>
```

Besides running Spark locally with multiple threads, we can also run
Spark on a [Spark standalone
cluster](https://spark.apache.org/docs/latest/spark-standalone.html).
To setup a Spark standalone cluster:
* Manually one by one
  1. Start a master:
  
     ```bash
     "${SPARK_DIR}"/sbin/start-master.sh
     ```
     
     It will output a `<master-spark-url>` like `spark://HOST:PORT`.
  
  1. Start workers and connect them to the master above:
  
     ```bash
     "${SPARK_DIR}"/sbin/start-slave.sh <master-spark-url>
     ```

* Or use cluster launch scripts

  1. Create a file called `conf/slaves` to spesify worker hostname.
     And optionally the configuration file called `conf/spark-env.sh`.
     **NOTE**: The master machine should be able SSH to each of the
     workers.
  
     ```bash
     # Run the command below on the master instead of local machine
     "${SPARK_DIR}"/sbin/start-all.sh
     ```

## Spark Configuration ##

[Spark
properties](https://spark.apache.org/docs/latest/configuration.html#spark-properties)
can be set via `SparkConf` programmatically, flags passed to
`spark-submit`/`spark-shell` or options in the
`conf/spark-defaults.conf` file with precedence from highest to
lowest.  Some properties may not be affected programmatically via
`SparkConf` in runtime, since they are related to deploy, such as
`spark.driver.memory`.

Besides using the default system Java and Python, Spark can also use
specified ones via [environment
variables](https://spark.apache.org/docs/latest/configuration.html#environment-variables)
set in the `conf/spark-env.sh` file, such as `JAVA_HOME` and
`PYSPARK_PYTHON`.


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

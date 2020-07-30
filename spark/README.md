# Spark #

## Getting Started ##

### Installation ###

1. Install prerequisites

   ```bash
   sudo apt update
   
   # Spark 2.4.x can only run on Java 8
   #     sudo apt install -y default-jre
   sudo apt install -y openjdk-8-jdk
   
   # Optional
   sudo update-alternatives --config java
   ```

1. [Install Spark](https://spark.apache.org/downloads.html).
    * It can be downloaded and installed as binaries:
        + Choose a Spark release: `2.4.5 (Feb 05 2020)`
        + Choose a package type: `Pre-built for Apache Hadoop 2.7`
        + Download Spark: `spark-2.4.5-bin-hadoop2.7.tgz`
    
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
      pyspark
      ```

### Run Spark Interactively ###

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

  ```bash
  # If Spark is installed as binaries, then
  sudo ln -s /usr/bin/python3 /usr/bin/python  # Optional
  cd "${SPARK_DIR}"
  "${SPARK_DIR}"/bin/pyspark
  
  # If Spark is installed as PySpark from PyPI, then
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

### Run Spark on Cluster ###

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


## Reference ##

* [Apache Spark](https://spark.apache.org/)
    + [Documentation](http://spark.apache.org/documentation.html)
        - [Latest Stable Version](https://spark.apache.org/docs/latest/index.html)
    + [Quick Start](https://spark.apache.org/docs/latest/quick-start.html)
    + [FAQ](https://spark.apache.org/faq.html)
    + [Building Spark](http://spark.apache.org/docs/latest/building-spark.html)
    + [Contributing to Spark](https://spark.apache.org/contributing.html)
    + [Useful Developer Tools](https://spark.apache.org/developer-tools.html)

# Spark #

## Getting Started ##

### Installation ###

1. Install prerequisites

   ```bash
   sudo apt update
   
   # Spark 2.4.x only support Java 8.
   # See [Spark Release 2.4.1](https://spark.apache.org/releases/spark-release-2-4-1.html).
   # Spark 3.x support Java 11 but does not support Scala 2.11 anymore.
   # See [Spark Release 3.0.0](https://spark.apache.org/releases/spark-release-3-0-0.html).
   sudo apt install -y openjdk-8-jdk
   
   # Optional
   sudo update-alternatives --config java
   
   # Python
   #
   # From 3.0.0 on, PySpark support Python 3.8.  PySpark of versions lower than 3.0.0 does not.
   # See [PySpark does not work with Python 3.8.0](https://issues.apache.org/jira/browse/SPARK-29536)
   # and the 'Programming Language' section on [PyPI](https://pypi.org/project/pyspark/)
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


## PySpark vs Spark ##

* Except `tests` and `testing`, `pyspark-3.0.0/pyspark/` is the same
  as `spark-3.0.0-bin-hadoop2.7/python/pyspark/` which contains Python
  code.
* `pyspark-3.0.0/deps/` is similar to `spark-3.0.0-bin-hadoop2.7/`:
    + `pyspark-3.0.0/deps/bin/` is the same as
      `spark-3.0.0-bin-hadoop2.7/bin/`
    + `pyspark-3.0.0/deps/jars/` is the same as
      `spark-3.0.0-bin-hadoop2.7/jars/`
    + `pyspark-3.0.0/deps/sbin/` lacks some scripts in
      `spark-3.0.0-bin-hadoop2.7/sbin/` for setting up Spark cluster,
      such as
        - `start-master.sh`, `stop-master.sh`
        - `start-slave.sh`, `stop-slave.sh`

So the tarball of PySpark is a restructured subset of Spark that lacks
the scripts for setting up cluster and Kubernetes.  The difference
between `spark-3.0.0-bin-hadoop2.7` and `spark-3.0.0-bin-hadoop3.2` is
the jars related to different versions of Hadoop.  In a nutshell,
[PySpark](https://github.com/apache/spark/tree/master/python) is
installation of [Spark](https://github.com/apache/spark) for running
Spark application in Python locally in a single node standalone mode
with multi-threads but not multi-nodes.

```console
$ tree -L 2 pyspark-3.0.0/
pyspark-3.0.0/
├── deps
│   ├── bin
│   ├── data
│   ├── examples
│   ├── jars
│   ├── licenses
│   └── sbin
├── lib
│   ├── py4j-0.10.9-src.zip
│   └── pyspark.zip
├── MANIFEST.in
├── PKG-INFO
├── pyspark
│   ├── accumulators.py
│   ├── broadcast.py
│   ├── cloudpickle.py
│   ├── conf.py
│   ├── context.py
│   ├── daemon.py
│   ├── files.py
│   ├── find_spark_home.py
│   ├── _globals.py
│   ├── heapq3.py
│   ├── __init__.py
│   ├── java_gateway.py
│   ├── join.py
│   ├── ml
│   ├── mllib
│   ├── profiler.py
│   ├── python
│   ├── rdd.py
│   ├── rddsampler.py
│   ├── resource.py
│   ├── resultiterable.py
│   ├── serializers.py
│   ├── shell.py
│   ├── shuffle.py
│   ├── sql
│   ├── statcounter.py
│   ├── status.py
│   ├── storagelevel.py
│   ├── streaming
│   ├── taskcontext.py
│   ├── traceback_utils.py
│   ├── util.py
│   ├── version.py
│   └── worker.py
├── pyspark.egg-info
│   ├── dependency_links.txt
│   ├── PKG-INFO
│   ├── requires.txt
│   ├── SOURCES.txt
│   └── top_level.txt
├── README.md
├── setup.cfg
└── setup.py

$ tree -L 1 spark-3.0.0-bin-hadoop2.7/
spark-3.0.0-bin-hadoop2.7/
├── bin
├── conf
├── data
├── examples
├── jars
├── kubernetes
├── LICENSE
├── licenses
├── NOTICE
├── python
├── R
├── README.md
├── RELEASE
├── sbin
└── yarn

$ tree -L 2 pyspark-3.0.0/deps/
pyspark-3.0.0/deps/
├── bin
│   ├── beeline
│   ├── beeline.cmd
│   ├── docker-image-tool.sh
│   ├── find-spark-home
│   ├── find-spark-home.cmd
│   ├── load-spark-env.cmd
│   ├── load-spark-env.sh
│   ├── pyspark
│   ├── pyspark2.cmd
│   ├── pyspark.cmd
│   ├── run-example
│   ├── run-example.cmd
│   ├── spark-class
│   ├── spark-class2.cmd
│   ├── spark-class.cmd
│   ├── sparkR
│   ├── sparkR2.cmd
│   ├── sparkR.cmd
│   ├── spark-shell
│   ├── spark-shell2.cmd
│   ├── spark-shell.cmd
│   ├── spark-sql
│   ├── spark-sql2.cmd
│   ├── spark-sql.cmd
│   ├── spark-submit
│   ├── spark-submit2.cmd
│   └── spark-submit.cmd
├── data
│   ├── graphx
│   ├── mllib
│   └── streaming
├── examples
├── jars
│   ├── ...
│   ├── hadoop-annotations-2.7.4.jar
│   ├── hadoop-auth-2.7.4.jar
│   ├── hadoop-client-2.7.4.jar
│   ├── hadoop-common-2.7.4.jar
│   ├── hadoop-hdfs-2.7.4.jar
│   ├── hadoop-mapreduce-client-app-2.7.4.jar
│   ├── hadoop-mapreduce-client-common-2.7.4.jar
│   ├── hadoop-mapreduce-client-core-2.7.4.jar
│   ├── hadoop-mapreduce-client-jobclient-2.7.4.jar
│   ├── hadoop-mapreduce-client-shuffle-2.7.4.jar
│   ├── hadoop-yarn-api-2.7.4.jar
│   ├── hadoop-yarn-client-2.7.4.jar
│   ├── hadoop-yarn-common-2.7.4.jar
│   ├── hadoop-yarn-server-common-2.7.4.jar
│   └── hadoop-yarn-server-web-proxy-2.7.4.jar
├── licenses
└── sbin
    ├── spark-config.sh
    ├── spark-daemon.sh
    ├── start-history-server.sh
    └── stop-history-server.sh

$ tree -L 2 spark-3.0.0-bin-hadoop2.7/
spark-3.0.0-bin-hadoop2.7/
├── bin
│   ├── beeline
│   ├── beeline.cmd
│   ├── docker-image-tool.sh
│   ├── find-spark-home
│   ├── find-spark-home.cmd
│   ├── load-spark-env.cmd
│   ├── load-spark-env.sh
│   ├── pyspark
│   ├── pyspark2.cmd
│   ├── pyspark.cmd
│   ├── run-example
│   ├── run-example.cmd
│   ├── spark-class
│   ├── spark-class2.cmd
│   ├── spark-class.cmd
│   ├── sparkR
│   ├── sparkR2.cmd
│   ├── sparkR.cmd
│   ├── spark-shell
│   ├── spark-shell2.cmd
│   ├── spark-shell.cmd
│   ├── spark-sql
│   ├── spark-sql2.cmd
│   ├── spark-sql.cmd
│   ├── spark-submit
│   ├── spark-submit2.cmd
│   └── spark-submit.cmd
├── conf
│   ├── fairscheduler.xml.template
│   ├── log4j.properties.template
│   ├── metrics.properties.template
│   ├── slaves.template
│   ├── spark-defaults.conf.template
│   └── spark-env.sh.template
├── data
│   ├── graphx
│   ├── mllib
│   └── streaming
├── examples
│   ├── jars
│   └── src
├── jars
│   ├── ...
│   ├── hadoop-annotations-2.7.4.jar
│   ├── hadoop-auth-2.7.4.jar
│   ├── hadoop-client-2.7.4.jar
│   ├── hadoop-common-2.7.4.jar
│   ├── hadoop-hdfs-2.7.4.jar
│   ├── hadoop-mapreduce-client-app-2.7.4.jar
│   ├── hadoop-mapreduce-client-common-2.7.4.jar
│   ├── hadoop-mapreduce-client-core-2.7.4.jar
│   ├── hadoop-mapreduce-client-jobclient-2.7.4.jar
│   ├── hadoop-mapreduce-client-shuffle-2.7.4.jar
│   ├── hadoop-yarn-api-2.7.4.jar
│   ├── hadoop-yarn-client-2.7.4.jar
│   ├── hadoop-yarn-common-2.7.4.jar
│   ├── hadoop-yarn-server-common-2.7.4.jar
│   └── hadoop-yarn-server-web-proxy-2.7.4.jar
├── kubernetes
│   ├── dockerfiles
│   └── tests
├── LICENSE
├── licenses
├── NOTICE
├── python
│   ├── dist
│   ├── docs
│   ├── lib
│   ├── MANIFEST.in
│   ├── pylintrc
│   ├── pyspark
│   ├── pyspark.egg-info
│   ├── README.md
│   ├── run-tests
│   ├── run-tests.py
│   ├── run-tests-with-coverage
│   ├── setup.cfg
│   ├── setup.py
│   ├── test_coverage
│   └── test_support
├── R
│   └── lib
├── README.md
├── RELEASE
├── sbin
│   ├── slaves.sh
│   ├── spark-config.sh
│   ├── spark-daemon.sh
│   ├── spark-daemons.sh
│   ├── start-all.sh
│   ├── start-history-server.sh
│   ├── start-master.sh
│   ├── start-mesos-dispatcher.sh
│   ├── start-mesos-shuffle-service.sh
│   ├── start-slave.sh
│   ├── start-slaves.sh
│   ├── start-thriftserver.sh
│   ├── stop-all.sh
│   ├── stop-history-server.sh
│   ├── stop-master.sh
│   ├── stop-mesos-dispatcher.sh
│   ├── stop-mesos-shuffle-service.sh
│   ├── stop-slave.sh
│   ├── stop-slaves.sh
│   └── stop-thriftserver.sh
└── yarn
    └── spark-3.0.0-yarn-shuffle.jar
```




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

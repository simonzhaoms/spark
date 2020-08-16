# PySpark #

In a nutshell,
[PySpark](https://github.com/apache/spark/tree/master/python) is
installation of [Spark](https://github.com/apache/spark) for running
Spark application in Python locally in a single node standalone mode
with multi-threads but not multi-nodes.  If a supported cluster is
already available, PySpark can be used to run Spark Application on the
cluster as well.


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
the jars related to different versions of Hadoop.

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

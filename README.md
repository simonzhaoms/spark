# Spark #

## Getting Started ##

1. [Download Apache Spark](https://spark.apache.org/downloads.html)
    * Choose a Spark release: `2.4.5 (Feb 05 2020)`
    * Choose a package type: `Pre-built for Apache Hadoop 2.7`
    * Download Spark: `spark-2.4.5-bin-hadoop2.7.tgz`
    
   ```bash
   SPARK_URL='https://downloads.apache.org/spark/spark-2.4.5/spark-2.4.5-bin-hadoop2.7.tgz'
   SPARK_FILE="${SPARK_URL##*/}"
   SPARK_DIR="${SPARK_FILE%\.*}"
   wget ${SPARK_URL}
   tar xf ${SPARK_FILE}
   ```

1. Install prerequisites

   ```bash
   sudo apt update
   
   # Spark 2.4.x can only run on Java 8
   #     sudo apt install -y default-jre
   sudo apt install -y openjdk-8-jdk
   
   # Optional
   sudo update-alternatives --config java
   ```

1. Run Spark shell:

    * Spark shell in Scala

      ```bash
      cd ${SPARK_DIR}
      ./bin/spark-shell
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
      sudo ln -s /usr/bin/python3 /usr/bin/python  # Optional
      cd ${SPARK_DIR}
      ./bin/pyspark
      ```

    * Spark shell in Python PySpark package

      ```bash
      sudo apt install -y python3-pip
      pip3 install pyspark
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


## Reference ##

* [Apache Spark](https://spark.apache.org/)
    + [Documentation](http://spark.apache.org/documentation.html)
        - [Latest Stable Version](https://spark.apache.org/docs/latest/index.html)
    + [Quick Start](https://spark.apache.org/docs/latest/quick-start.html)
    + [FAQ](https://spark.apache.org/faq.html)
    + [Building Spark](http://spark.apache.org/docs/latest/building-spark.html)
    + [Contributing to Spark](https://spark.apache.org/contributing.html)
    + [Useful Developer Tools](https://spark.apache.org/developer-tools.html)

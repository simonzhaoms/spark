# Environment Setup #

## Prerequisites ##

**TODO**: Now Scala and Spark support Java 11.  See [JDK
Compatibility](https://docs.scala-lang.org/overviews/jdk-compatibility/overview.html)
and [Spark Release
3.0.0](https://spark.apache.org/releases/spark-release-3-0-0.html)

* Java 8: Scala runs on JVM

  1. Install [SDKMAN](https://sdkman.io/): A SDK manager like Conda
     for easily switching between different versions of SDKs, such as
     OpenJDK and Oracle Java
  
     ```bash
     sudo apt install -y curl
     curl -s "https://get.sdkman.io" | bash
     ```
     
     ```bash
     source "$HOME/.sdkman/bin/sdkman-init.sh"
     
     ```

  1. Install Java 8: Seems Spark is not compatible with higher version
     of Java

     ```bash
     sdk list java
     sdk install java 8.0.252.hs-adpt
     sdk default java 8.0.252.hs-adpt
     ```


## REPL ##

### Basic Scala interactive interpreter ###

1. Install Java 8
1. [Download Scala](https://www.scala-lang.org/download/)
1. Decompress and run `./bin/scala` in the decompressed folder


### [Ammonite](http://ammonite.io/) (**TODO**) ###



## IDE (IntelliJ) ##

1. Install Java 8
1. Install [IntelliJ IDEA](https://www.jetbrains.com/idea/download/)

   ```bash
   sudo snap install intellij-idea-community --classic
   ```


## Command Line (**TODO**) ##

1. Install Java 8 and [sbt](https://www.scala-sbt.org)

   ```bash
   sdk install java 8.0.252.hs-adpt
   sdk default java 8.0.252.hs-adpt
   sdk install sbt
   ```

1. Create a project

   ```bash
   mkdir ${PROJECT_NAME}
   cd ${PROJECT_NAME}
   sbt new scala/hello-world.g8
   ```

## Reference ##

* [Getting Started](https://docs.scala-lang.org/getting-started/index.html)
    + [Download](https://www.scala-lang.org/download/)
    + [Getting started with Scala in IntelliJ](https://docs.scala-lang.org/getting-started/intellij-track/getting-started-with-scala-in-intellij.html)
    + [Getting started with Scala and sbt on the command line](https://docs.scala-lang.org/getting-started/sbt-track/getting-started-with-scala-and-sbt-on-the-command-line.html)
* [SDKMAN](https://sdkman.io/)
    + [Usage](https://sdkman.io/usage)

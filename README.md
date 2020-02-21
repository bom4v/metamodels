Build of Business-oriented Object Models (BOM)
==============================================

[![Docker Repository on Quay](https://quay.io/repository/bom4v/metamodels/status "Docker Repository on Quay")](https://quay.io/repository/bom4v/metamodels)

# Overview
Business-relted Object Models (BOM) are software classes (incarned here
through Scala case classes) modelling the business of a particular industry
(here, an example is given for the telecom industry, but it can easily be
generalized to other industries). For instance, there are software classes
representing customers, customer accounts, interactions (_e.g._, calls,
messages) that customers exchange among them, markets, and so on.

The fields of those software classes, in addition to their respective
associated methods (_e.g._, classical data processing functions such as
aggregation and filtering), make up a high level API (Application
Programming Interface).
The software developers, data engineers and other data scientists
can then reasonate at the business level when implementing more
complex data processing techniques.
For instance, in order to calculate the distribution of
the call duration between any two customers of a given market,
a developer has just to invoke the corresponding method on
the Scala case class representing a customer.

For each business-related software class, there are associated
serializers and de-serializers, allowing:
* On one hand, to parse any raw data (_e.g._, call data records (CDR)
  in a standard `NRT ASN.1` binary format) and fill in the fields
  of those classes
* On the other hand, to dump the resulting data into flat CSV files

Then, the output CSV files can either be (in a non exclusive manner):
* Uploaded into any classical or NoSQL databases for further querying
  by data analysts and data scientists
* Served through a Web Service (WS) API, enabling business
  and data analysts to interact with on-line analytics almost
  out-of-the-box (for instance thanks to
  [MS PowerBI](https://powerbi.microsoft.com),
  [Splunk](http://splunk.com) or [Tableau](http://tableau.com))

Java artefacts (JAR files) are produced (built) for each component,
and can then be published in artefact repositories (_e.g._,
[Maven](https://maven.apache.org), [Nexus](https://repository.apache.org)),
for subsequent delivery onto Spark-based production systems,
either on-premises or on clouds (eg, DataBricks, GCP, AWS or Azure).
The release versions of the BOM4V artefacts are stored on
the so-called
[Maven Central repository](https://repo1.maven.org/maven2/org/bom4v/ti/),
while the snapshot versions are stored on the
[OSS Sonatype repository](https://oss.sonatype.org/content/repositories/snapshots/org/bom4v/ti). 

The `metamodels` project itself is an umbrella, allowing to drive all
the other components from a central local directory, namely `workspace/src`.
One can then interact with any specific component directly by jumping
(`cd`-ing) into the corresponding directory. Software code can be edited
and committed directly from that component sub-directory.

[Docker images, hosted on Docker Cloud](https://cloud.docker.com/u/bom4v/repository/docker/bom4v/sparkml),
are provided for convenience reason, avoiding the need to set up
a proper development environment: they provide ready-to-use,
ready-to-develop, ready-to-contribute environments on top of
a few well known Linux distributions
(_e.g._, [CentOS 7](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7),
[Debian 9 (Stretch)](https://www.debian.org/releases/stretch/) and
[Ubuntu 18.10 (Cosmic Cuttlefish)](http://releases.ubuntu.com/18.10/)).
Enjoy!

# References
* Docker Cloud repository:
  https://cloud.docker.com/u/bom4v/repository/docker/bom4v/sparkml
  + Base Docker images:
    https://cloud.docker.com/u/bigdatadevelopment/repository/docker/bigdatadevelopment/base
* GitHub organizations:
  + Business-oriented Object Models (BOM) for the TI vertical (BOM4V):
    http://github.com/bom4v
  + Telecoms Intelligence (TI): http://github.com/telecomsintelligence
  + Transport Intelligence (TI): http://github.com/transport-intelligence
  + Travel Intelligence (TI): http://github.com/travel-intelligence

# Run the Docker image
* As a quick starter, a Spark-based churn detection example may be run
  from any of
  [Docker images](https://cloud.docker.com/u/bom4v/repository/docker/bom4v/sparkml),
  where `<linux-distrib>` is `centos`, `debian` or `ubuntu`:
```bash
$ docker run --rm -it bom4v/sparkml:<linux-distrib> bash
[build@c..5 bom4v]$ cd workspace/src/ti-spark-examples
[build@c..5 ti-spark-examples (master)]$ ./fillLocalDataDir.sh
[build@c..5 ti-spark-examples (master)]$ sbt "runMain org.bom4v.ti.Demonstrator"
[info] ...
root
 |-- specificationVersionNumber: integer (nullable = true)
 ...
 |-- servingNetwork: string (nullable = true)

+-----+-----+                                                                   
|churn|count|
+-----+-----+
|False| 2278|
| True|  388|
+-----+-----+

+-----+-----+                                                                   
|churn|count|
+-----+-----+
|False|  379|
| True|  388|
+-----+-----+

area under the precision-recall curve: 0.9747578698231796
area under the receiver operating characteristic (ROC) curve : 0.8484817813765183

counttotal : 667
correct : 574
wrong: 93
ratio wrong: 0.13943028485757122
ratio correct: 0.8605697151424287
ratio true positive : 0.1184407796101949
ratio false positive : 0.0239880059970015
ratio true negative : 0.7421289355322339
ratio false negative : 0.11544227886056972

[success] Total time: 63 s, completed Dec 19, 2018 4:03:30 PM
[build@c..5 ti-spark-examples (master)]$ exit
```

# Meta-build
The
[Docker images](https://cloud.docker.com/u/bom4v/repository/docker/bom4v/sparkml)
come with all the dependencies already installed. If there is a need,
however, for some more customization (for instance, other software products
such as [Kafka](https://kafka.apache.org) or
[ElasticSearch](http://elasticsearch.com)), this section describes
how to get the end-to-end Spark-based churn prediction example up
and running on a native environment (as opposed to within
a Docker container).

An alternative is to develop your own Docker image from the
[one provided by that project](https://cloud.docker.com/u/bom4v/repository/docker/bom4v/sparkml).
You would typically start the `Dockerfile` with
`FROM bom4v/sparkml:<linux-distrib>`, where `<linux-distrib>` is `centos`,
`debian` or `ubuntu`.

## Installation of dependencies (if not using the Docker image)
[Java](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
and [Scala](https://www.scala-lang.org/download/) are needed in order to build
and run the various components of that project. Moreover, the current execution
engines currently relies on [Spark](https://spark.apache.org).

## Docker images
The
[maintained Docker images for that project](http://github.com/bom4v/sparkml/tree/master/docker/)
come with all the necessary pieces of software. They can either be used
_as is_, or used as inspiration for _ad hoc_ setup on other configurations.

## Native environment (outside of Docker)

### CentOS/RedHat
* Install [EPEL for CentOS/RedHat](https://fedoraproject.org/wiki/EPEL):
```bash
$ sudo rpm --import http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7
$ sudo yum -y install epel-release
```

* Install a few useful packages:
```bash
sudo yum -y install less htop net-tools which sudo man vim \
	git-all wget curl file bash-completion keyutils \
	gzip tar unzip maven rake rubygem-rake rubygem-nokogiri
```

* Install [SBT](https://www.scala-sbt.org/download.html):
```bash
$ sudo (curl -sS https://bintray.com/sbt/rpm/rpm | \
	tee /etc/yum.repos.d/bintray-sbt-rpm.repo)
$ sudo yum -y install sbt
```

* Install the [Java 8 OpenJDK](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html):
```bash
$ sudo wget -c --header "Cookie: oraclelicense=accept-securebackup-cookie" \
	http://download.oracle.com/otn-pub/java/jdk/8u192-b12/750e1c8617c5452694857ad95c3ee230/jdk-8u192-linux-x64.rpm && \
	yum -y install jdk-8u192-linux-x64.rpm && rm -f jdk-8u192-linux-x64.rpm
```

* Download and install Scala:
```bash
$ sudo SCALA_VERSION=2.11.8 && \
	mkdir -p /opt/scala && \
	wget http://www.scala-lang.org/files/archive/scala-$SCALA_VERSION.tgz && \
	tar xvf scala-$SCALA_VERSION.tgz && \
	mv scala-$SCALA_VERSION /opt/scala && \
	rm -f /opt/scala/latest && \
	ln -s /opt/scala/scala-$SCALA_VERSION /opt/scala/latest && \
	rm -f scala-$SCALA_VERSION.tgz
$ cat >> ~/.bashrc << _EOF

# Scala
export PATH=\${PATH}:/opt/scala/latest/bin
_EOF
$ . ~/.bashrc
```

### MacOS
* Java 8, Maven, SBT and Apache Spark:
```bash
$ brew tap adoptopenjdk/openjdk
$ brew cask install adoptopenjdk8
$ brew install maven sbt scala apache-spark
```

### All
* Run SBT once, as to download and cache all the dependencies:
```bash
$ scala -version
Scala code runner version 2.11.8 -- Copyright 2002-2016, LAMP/EPFL
$ sbt about
[info] Loading project definition from ~/project
[info] This is sbt 1.2.7
[info] The current project is built against Scala 2.12.7
```

## Clone the Git repository
The following operation needs to be done only on a native environment (as
opposed to within a Docker container).
The Docker image indeed comes with that Git repository already cloned and built.
```bash
$ mkdir -p ~/dev/bom4v && cd ~/dev/bom4v
$ git clone https://github.com/bom4v/metamodels.git
$ cd metamodels
$ cp docker/centos/resources/metamodels.yaml.sample metamodels.yaml
$ ln -s docker/centos/resources/Rakefile Rakefile
$ rake clone
$ rake checkout
$ rake offline=true info
```

## Interactive build with `rake`
That operation may be done either from within the Docker container,
or in a native environment (on which the dependencies have been installed).

As a reminder, to enter into the container, just type
`docker run --rm -it bom4v/sparkml:<linux-distrib> bash` (and `exit` to leave it).

The following sequence of commands describes how to build, test and deliver
the artefacts of all the components, so that Spark can execute the full project:
```bash
$ cd ~/dev/bom4v/metamodels
$ rm -rf ~/.m2 ~/.ivy2
$ rake offline=true deliver
$ rake offline=true test
```

## Interacting with a specific project
Those operations may be done either from within the Docker container,
or in a native environment (on which the dependencies have been installed).

As a reminder, to enter into the container, just type
`docker run --rm -it bom4v/sparkml:<linux-distrib> bash`,
where `<linux-distrib>` is `centos`, `debian` or `ubuntu`
(and `exit` to leave it).

```bash
$ cd ~/dev/bom4v/metamodels
$ cd workspace/src/ti-models-calls
$ vi src/main/scala/org/bom4v/ti/models/calls/CallsModel.scala
$ git add src/main/scala/org/bom4v/ti/models/calls/CallsModel.scala
$ sbt +compile +test
$ sbt 'set isSnapshot := true' +publish-local +publish-m2
$ cd -
$ cd workspace/src/ti-spark-examples
$ vi src/main/scala/org/bom4v/ti/Demonstrator.scala
$ git add src/main/scala/org/bom4v/ti/Demonstrator.scala
$ sbt +compile +test
$ cd -
$ rake offline=true test
$ # If all goes well at the integration level
$ cd workspace/src/ti-models-calls
$ git commit -m "[Dev] Fixed issue #76: wrong field type for the call number"
$ cd -
$ cd workspace/src/ti-spark-examples
$ git commit -m "[Dev] Adapted to the new ti-models-calls structure"
$ cd -
```

## Batched build and Docker image generation
If the Docker images need to be re-built, the following commands explain
how to do it:
```bash
$ mkdir -p ~/dev/bom4v && cd ~/dev/bom4v
$ git clone https://github.com/bom4v/metamodels.git
$ cd metamodels
$ docker build -t bom4v/sparkml:<linux-distrib> --squash docker/<linux-distrib>/
$ docker push bom4v/sparkml:<linux-distrib>
$ docker images | grep "^bom4v"
REPOSITORY      TAG             IMAGE ID        CREATED             SIZE
bom4v/sparkml   <linux-distrib> 9a33eee22a3d    About an hour ago   2.16GB
```

# Interacting with a Spark installation
So far, we have seen how to launch the application on the Spark engine
embedded by the JVM spawned by SBT. That embedded Spark engine has
[some limitations](https://stackoverflow.com/questions/44298847/why-do-we-need-to-add-fork-in-run-true-when-running-spark-sbt-application),
and a
[vanilla version of Spark installation](https://spark.apache.org/downloads.html)
may be preferred for more demanding use cases.

On recent Spark installations, there is no need to prefix
file-paths by `hdfs://` or to specify absolute file-paths:
* In stand-alone mode, Spark will look in the local file-system
* In cluster mode, Spark will look in HDFS. If the file-paths
  are relative, then Spark will look relatively from the
  user home directory (typically, `/user/$USER`) on HDFS

In the following sections, details are given on how to interact
with HDFS for instance, to transfer back and forth betwwen
the local filesystem and HDFS), but most of those operations
are now optional on a local Spark installation.

## (Optional) Copy the data onto HDFS
```bash
$ export HDFS_URL="hdfs://127.0.0.1:9000"
$ alias hdfsfs='hdfs dfs -Dfs.defaultFS=$HDFS_URL'
$ export HDFS_USR_DIR="/user/<user>"
$ hdfsfs -mkdir -p $HDFS_USR_DIR/data/cdr
$ hdfsfs -put data/cdr/CDR-sample.csv $HDFS_USR_DIR/data/cdr
$ hdfsfs -cat $HDFS_USR_DIR/data/cdr/CDR-sample.csv|head -3
```

## Local Spark cluster
* It is assumed here that
  [Spark has been installed locally](https://spark.apache.org/downloads.html)
```bash
$ export MVN_CHD_REPO="$HOME/.m2/repository"
$ $SPARK_HOME/bin/spark-submit \
  --class org.bom4v.ti.Demonstrator \
  --master local --deploy-mode client \
  --jars \
file:$MVN_CHD_REPO/org/bom4v/ti/ti-models-calls_2.11/0.0.1/ti-models-calls_2.11-0.0.1.jar,\
file:$MVN_CHD_REPO/org/bom4v/ti/ti-serializers-calls_2.11/0.0.1-spark2.3/ti-serializers-calls_2.11-0.0.1-spark2.3.jar,\
file:$MVN_CHD_REPO/org/bom4v/ti/ti-serializers-customers_2.11/0.0.1-spark2.3/ti-serializers-customers_2.11-0.0.1-spark2.3.jar,\
file:$MVN_CHD_REPO/org/bom4v/ti/ti-models-customers_2.11/0.0.1/ti-models-customers_2.11-0.0.1.jar \
  target/scala-2.11/ti-spark-examples_2.11-0.0.1-spark2.3.jar
```

## Spark cluster - Client mode
* It is assumed here that a Spark cluster has been installed
  somewhere, and that you are allowed to launch jobs on that
  cluster
* On some recent local installations of Spark, for instance
  on MacOS, the Yarn cluster client mode is equivalent to
  the local mode
```bash
$ $SPARK_HOME/bin/spark-submit \
  --class org.bom4v.ti.Demonstrator \
  --master yarn --deploy-mode client \
  --jars \
file:$MVN_CHD_REPO/org/bom4v/ti/ti-models-calls_2.11/0.0.1/ti-models-calls_2.11-0.0.1.jar,\
file:$MVN_CHD_REPO/org/bom4v/ti/ti-serializers-calls_2.11/0.0.1-spark2.3/ti-serializers-calls_2.11-0.0.1-spark2.3.jar,\
file:$MVN_CHD_REPO/org/bom4v/ti/ti-serializers-customers_2.11/0.0.1-spark2.3/ti-serializers-customers_2.11-0.0.1-spark2.3.jar,\
file:$MVN_CHD_REPO/org/bom4v/ti/ti-models-customers_2.11/0.0.1/ti-models-customers_2.11-0.0.1.jar \
  target/scala-2.11/ti-spark-examples_2.11-0.0.1-spark2.3.jar
```

## Spark cluster - Server mode
If the jobs are to be launched from a remote machine, you may want to map the local HDFS port
to the HDFS port of the remote machine. For instance, from an independent terminal window
on the local machine:
```bash
$ The -N option allows to not launch any command (eg, bash)
$ ssh <user>@<remote-machine> -N -L 9000:127.0.0.1:9000
```

Then, the following commands will work:
* remotely if the above SSH port forwarding has been set up
* locally if the above SSH port forwarding has not been set up
```bash
$ export HDFS_URL="hdfs://127.0.0.1:9000"
$ alias hdfsfs='hdfs dfs -Dfs.defaultFS=${HDFS_URL}'
$ export ATF_USR_DIR="/user/<user>/artefacts"
$ export ATF_USR_URL="${HDFS_URL}${ATF_USR_DIR}"
$ hdfsfs -mkdir -p $ATF_USR_DIR
$ hdfsfs -put -f target/scala-2.11/ti-spark-examples_2.11-0.0.1-spark2.3.jar $ATF_USR_DIR
```

```bash
$ $SPARK_HOME/bin/spark-submit \
  --class org.bom4v.ti.Demonstrator \
  --master yarn --deploy-mode cluster \
  --jars \
file:$MVN_CHD_REPO/org/bom4v/ti/ti-models-calls_2.11/0.0.1/ti-models-calls_2.11-0.0.1.jar,\
file:$MVN_CHD_REPO/org/bom4v/ti/ti-serializers-calls_2.11/0.0.1-spark2.3/ti-serializers-calls_2.11-0.0.1-spark2.3.jar,\
file:$MVN_CHD_REPO/org/bom4v/ti/ti-serializers-customers_2.11/0.0.1-spark2.3/ti-serializers-customers_2.11-0.0.1-spark2.3.jar,\
file:$MVN_CHD_REPO/org/bom4v/ti/ti-models-customers_2.11/0.0.1/ti-models-customers_2.11-0.0.1.jar \
  target/scala-2.11/ti-spark-examples_2.11-0.0.1-spark2.3.jar
```


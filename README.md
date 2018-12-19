Build of Business-oriented Object Models (BOM)
==============================================

# Overview
Business-relted Object Models (BOM) are software classes (incarned here
through Scala case classes) modelling the business of a particular industry
(here, an example is given for the telecom industry, but it can easily be
generalized to other industries). For instance, there are software classes
representing customers, customer accounts, interactions (_e.g._, calls,
messages) that customers exchange among them, markets, and so on.

The fields of those software classes, in addition to their respective
associated methods (eg, classical data processing functions such as
aggregation and filtering), make up a high level API. The software
developers, data engineers and other data scientists can then reasonate
at the business level when implementing more complex data processing
techniques. For instance, in order to calculate the distribution of
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

The `metamodels` project itself is an umbrella, allowing to drive all
the other components from a central local directory, namely `workspace/src`.
One can then interact with any specific component directly by jumping
(`cd`-ing) into the corresponding directory. Software code can be edited
and committed directly from that component sub-directory.

[Docker images, hosted on Docker Cloud](https://cloud.docker.com/u/bom4v/repository/docker/bom4v/sparkml),
are provided for convenience reason, avoiding the need to set up
a proper development environment: they provide a ready-to-use,
ready-to-develop, ready-to-contribute environment. Enjoy!

# References
* Docker Cloud repository: https://cloud.docker.com/u/bom4v/repository/docker/bom4v/sparkml
* GitHub organizations:
  + Business-oriented Object Models (BOM) for the TI vertical (BOM4V):
    http://github.com/bom4v
  + Telecoms Intelligence (TI): http://github.com/telecomsintelligence
  + Transport Intelligence (TI): http://github.com/transport-intelligence
  + Travel Intelligence (TI): http://github.com/travel-intelligence

# Run the Docker image
* As a quick starter, a Spark-based churn detection example may be run
  from the [`sparkml` Docker image]():
```bash
$ docker run --rm -it bom4v/sparkml:latest bash
[build@c..5 bom4v]$ cd workspace/src/ti-spark-examples
[build@c..5 ti-spark-examples (master)]$ ./fillLocalDataDir.sh
[build@c..5 ti-spark-examples (master)]$ sbt run
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
The [Docker image](https://cloud.docker.com/u/bom4v/repository/docker/bom4v/sparkml)
come with all the dependencies already installed. If there is a need,
however, for some more customization (for instance, having a different
Linux distribution than [CentOS](http://centos.org), or install some
other software products such as [Kafka](https://kafka.apache.org) or
[ElasticSearch](http://elasticsearch.com)), this section describes
how to get the end-to-end Spark-based churn prediction example up
and running on a native environment (as opposed to within
a Docker container).

An alternative is to develop your own Docker image from the
[one provided by that project](https://cloud.docker.com/u/bom4v/repository/docker/bom4v/sparkml).
You would typically start the `Dockerfile` with `FROM bom4v/sparkml:base`
or `FROM bom4v/sparkml:latest`.

## Installation of dependencies (if not using the Docker image)
[Java](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
and [Scala](https://www.scala-lang.org/download/) are needed in order to build
and run the various components of that project. Moreover, the current execution
engines currently relies on [Spark](https://spark.apache.org).

## Docker images
The [maintained Docker image for that project](docker/base/)
comes with all the necessary pieces of software. They can either be used
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
$ brew cask install homebrew/cask-versions/java8
$ export JAVA_HOME=`/usr/libexec/java_home -v 1.8`
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
$ cp docker/builder/resources/metamodels.yaml.sample metamodels.yaml
$ ln -s docker/builder/resources/Rakefile Rakefile
$ rake clone
$ rake checkout
$ rake offline=true info
```

## Interactive build with `rake`
That operation may be done either from within the Docker container,
or in a native environment (on which the dependencies have been installed).

As a reminder, to enter into the container, just type
`docker run --rm -it bom4v/sparkml:latest bash` (and `exit` to leave it).

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
`docker run --rm -it bom4v/sparkml:latest bash` (and `exit` to leave it).

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
$ docker build -t bom4v/sparkml:base --squash docker/base/
$ docker push bom4v/sparkml:base
$ docker build -t bom4v/sparkml:beta --squash docker/sparkml/
$ docker push bom4v/sparkml:beta
$ docker images | grep "^bom4v"
REPOSITORY       TAG        IMAGE ID        CREATED             SIZE
bom4v/sparkml    beta       9a33eee22a3d    About an hour ago   2.16GB
bom4v/sparkml    base       c5f1ea63a79b    2 hours ago         1.66GB
```


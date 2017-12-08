Build of Telecoms Intelligence (TI) Business-related Object Models
==================================================================

# Overview
Business-relted Object Models (BOM) are software classes (incarned here
through Scala case classes) modelling the business of a particular industry
(here, the telecom industry). For instance, there are software classes
representing customers, customer accounts, interactions (eg, calls,
messages) that customers exchange among them, markets, and so on.

The fields of those software classes, in addition to their respective
associated methods (eg, classical data processing functions such as aggregation
and filtering), make up a high level API. The software
developers, data engineers and other data scientists can then reasonate
at the business level when implementing more complex data processing
techniques. For instance, in order to calculate the distribution of
the call duration between any two customers of a given market,
a developer has just to invoke the corresponding method on the customer
class.

For each business-related software class, there are associated serializers
and de-serializers, allowing:
* On one hand, to parse any raw data (eg, CDR in a standard
NRT ASN.1 binary format) and fill in the fields of those classes
* On the other hand, to dump the resulting data into flat CSV files

Then, the output CSV files can either be (in a non exclusive manner):
* Uploaded into any classical or NoSQL databases for further querying
by data analysts and data scientists
* Served through a RESTful API, enabling business and data analysts
to interact with on-line analytics almost out-of-the-box (for instance
thanks to MS PowerBI, Splunk or Tableau)

The projects generate Java artefacts (JAR files), which can then be
published to artefact repositories (eg, Maven, Nexus), and be delivered
onto Spark production systems, either on-premises or on clouds (eg,
DataBricks, GCP, AWS or Azure).

The ``metamodels`` project itself is an umbrella, allowing to drive all
the other projects from a central local directory, namely ``workspace/src``.
One can then interact with any specific project directly by jumping
(``cd``-ing) into the corresponding directory. Software code can be edited
and committed directly from that project sub-directory.

# References
* Docker Hub repository: https://hub.docker.com/r/telecomsintelligence/bom4v
* GitHub organizations:
  * Telecoms Intelligence (TI): http://github.com/telecomsintelligence
  * Business-oriented Object Models (BOM) for the TI vertical (BOM4V): http://github.com/bom4v

# Meta-build
## Interactive build with ``rake``
```bash
$ mkdir -p ~/dev/ti/ti-models
$ cd ~/dev/ti/ti-models
$ git clone https://github.com/bom4v/metamodels.git
$ cd metamodels
$ cp metamodels.yaml.sample metamodels.yaml
$ rake clone
$ rake checkout
$ rake offline=true info
$ rm -rf ~/.m2 ~/.ivy2
$ rake offline=true deliver
$ rake offline=true test
```

## Interacting with a specific project
```bash
$ cd ~/dev/ti/ti-models/metamodels
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
```bash
$ mkdir -p ~/dev/ti/ti-models
$ cd ~/dev/ti/ti-models
$ git clone https://github.com/bom4v/metamodels.git
$ cd metamodels
$ docker build -t telecomsintelligence/bom4v:latest --squash .
$ docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
telecomsintelligence/bom4v      latest              981cb5f04428        20 seconds ago      1.3GB
$ docker push telecomsintelligence/bom4v:latest
```

# Run the Docker image
```bash
$ docker run --rm -it telecomsintelligence/bom4v:latest bash
$ cd workspace/src/ti-spark-examples
$ ./mkLocalDir.sh
$ sbt run
```


Build of Telecoms Intelligence (TI) Business-related Object Models
==================================================================

# References
* Docker Hub repository: https://hub.docker.com/r/telecomsintelligence/bom4v
* GitHub organizations:
  * Telecoms Intelligence (TI): http://github.com/telecomsintelligence
  * Business-oriented Object Models (BOM) for the TI vertical (BOM4V): http://github.com/bom4v

# Meta-build
## Inteeractive build with ``rake``
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


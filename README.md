
# Meta-build
## Inteeractive (with ``rake``)
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

## By batch (with Docker)
```bash
$ mkdir -p ~/dev/ti/ti-models
$ cd ~/dev/ti/ti-models
$ git clone https://github.com/bom4v/metamodels.git
$ cd metamodels
$ docker build -t telecomsintelligence/bom4v:latest --squash .
$ docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
telecomsintelligence/bom4v      latest              981cb5f04428        20 seconds ago      1.3GB
```

# Run the Docker image
```bash
$ docker run -i telecomsintelligence/bom4v:latest bash
```


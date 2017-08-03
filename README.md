
# Meta-build
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


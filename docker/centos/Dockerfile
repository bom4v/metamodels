#
# Dockerfile: https://github.com/bom4v/metamodels/tree/master/docker/centos/Dockerfile
# Docker Hub/Cloud page: https://cloud.docker.com/u/infrahelpers/repository/docker/infrahelpers/bom4v
# Usual Docker tag: infrahelpers/bom4v:centos
#
# Umbrella project for the BOM for Verticals (BOM4V) components
#
# Base image:
# * Usual Docker tag: infrahelpers/javabigdata:centos8
# * Docker Hub/Cloud page: https://cloud.docker.com/u/infrahelpers/repository/docker/infrahelpers/javabigdata
# * Dockerfile: https://github.com/bom4v/docker-images/tree/master/centos8/Dockerfile
#
FROM infrahelpers/javabigdata:centos8

LABEL authors="Denis Arnaud <denis.arnaud_github at m4x dot org>"

# Docker build time environment variables
ENV container docker
ENV HOME /home/build
ENV LANGUAGE en_US:en
ENV LANG en_US.UTF-8
ENV LC_ALL $LANG
ENV BOM4V_DIR $HOME/dev/bom4v

# Switch to the `build` user
USER build
WORKDIR $BOM4V_DIR

# Prepare the build environment
ADD resources/Rakefile resources/metamodels.yaml.sample $BOM4V_DIR/
RUN mv $BOM4V_DIR/metamodels.yaml.sample $BOM4V_DIR/metamodels.yaml

# Build the BOM for Verticals (BOM4V) components
RUN source $HOME/.bashrc && \
	rake clone && rake checkout && \
	rake offline=true deliver

# Entry point
CMD ["/bin/bash"]


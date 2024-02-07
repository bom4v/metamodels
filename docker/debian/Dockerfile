#
# Dockerfile: https://github.com/bom4v/metamodels/tree/master/docker/debian/Dockerfile
# Docker Hub/Cloud page: https://cloud.docker.com/u/infrahelpers/repository/docker/infrahelpers/bom4v
# Usual Docker tag: infrahelpers/bom4v:debian
#
# Umbrella project for the BOM for Verticals (BOM4V) components
#
# Base image:
# * Usual Docker tag: infrahelpers/javabigdata:debian12
# * Docker Hub/Cloud page: https://cloud.docker.com/u/infrahelpers/repository/docker/infrahelpers/javabigdata
# * Dockerfile: https://github.com/bom4v/docker-images/tree/master/debian12/Dockerfile
#
FROM infrahelpers/javabigdata:debian12

LABEL authors="Denis Arnaud <denis.arnaud_github at m4x dot org>"

# Docker build time environment variables
ENV container docker
ENV HOME /home/build
ENV LANGUAGE en_US:en
ENV LANG en_US.UTF-8
ENV LC_ALL $LANG
ENV BOM4V_DIR $HOME/dev/bom4v

# Install a few Ruby packages and the nokogiri Gem
USER root
RUN apt-get -qq update && \
    apt-get -y install zlib1g-dev libbz2-dev \
        lsb-release libgmp-dev \
        gcc g++ cppcheck clang cmake manpages patch pkg-config \
        m4 autoconf automake libtool libltdl-dev build-essential \
        ruby ruby-dev rubygems

# Switch to the `build` user
USER build
WORKDIR $BOM4V_DIR

# Prepare the build environment
ADD resources/Rakefile resources/metamodels.yaml.sample $BOM4V_DIR/
RUN mv $BOM4V_DIR/metamodels.yaml.sample $BOM4V_DIR/metamodels.yaml

# Build the BOM for Verticals (BOM4V) components
ENV PATH $PATH:/usr/lib/scala/bin
RUN rake clone && rake checkout && \
	rake offline=true deliver

# Entry point
CMD ["/bin/bash"]


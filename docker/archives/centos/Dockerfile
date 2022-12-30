#
# http://github.com/bom4v/metamodels/tree/master/docker/centos
# Umbrella project for the BOM for Verticals (BOM4V) components
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

# Install development packages for Ruby
USER root
RUN dnf -y install zlib-devel make m4 automake autoconf libtool \
    langtable gcc gcc-c++ \
    redhat-rpm-config \
    ruby-devel rubygems-devel

# Install the nokogiri Gem
RUN gem install nokogiri --platform=ruby

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


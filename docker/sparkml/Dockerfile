#
# http://github.com/bom4v/metamodels/tree/master/docker/sparkml
#

FROM bom4v/sparkml:base
MAINTAINER Denis Arnaud <denis.arnaud_github at m4x dot org>

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

# Build the TI Models
RUN export PATH=$PATH:/usr/lib/scala/bin && \
	rake clone && \
	rake checkout && \
	rake offline=true deliver

# Entry point
CMD ["/bin/bash"]


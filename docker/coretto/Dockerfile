#
# http://github.com/bom4v/metamodels/tree/master/docker/coretto
# Umbrella project for the BOM for Verticals (BOM4V) components
#
# Container image specialized for deployment of Spark jobs on AWS EMR/Databricks
#
# Inspired by: https://aws.amazon.com/blogs/big-data/simplify-your-spark-dependency-management-with-docker-in-emr-6-0-0
#
# See also:
#  + Base image Dockerfile: https://github.com/cloud-helpers/cloud-python-images/blob/main/pyspark-emr-6-coretto-8/Dockerfile
#  + Base image on Docker Hub: https://hub.docker.com/repository/docker/infrahelpers/cloud-python/general
#     - Usual Docker tag: infrahelpers/cloud-python:pyspark-emr6
#  + Coretto:
#    - https://docs.aws.amazon.com/corretto/latest/corretto-8-ug/what-is-corretto-8.html
#    - https://docs.aws.amazon.com/corretto/latest/corretto-11-ug/what-is-corretto-11.html
#    - https://docs.aws.amazon.com/corretto/latest/corretto-17-ug/docker-install.html
# The underlying operating system (OS) is Amazon Linux 2, i.e., based on a
# RedHat Linux 7 with some Amazon specific addition.
# EMR, up to version at least 6.5.0, uses Java 8 internally (according to
# https://docs.aws.amazon.com/emr/latest/ReleaseGuide/configuring-java8.html)
#
FROM infrahelpers/cloud-python:pyspark-emr-dbs

LABEL authors "Denis Arnaud <denis.arnaud_fedora@m4x.org>"

# Docker build time environment variables
ENV container docker
ENV LANGUAGE en_US:en
ENV LANG en_US.UTF-8
ENV LC_ALL $LANG
ENV RUBY_VERSION 3.2.0
ENV HOME /home/build
ENV BOM4V_DIR $HOME/dev/bom4v

# Install development packages for Ruby
RUN yum -y install zlib-devel bzip2-devel gzip tar libyaml-devel \
	make m4 autoconf automake libtool \
    langtable gcc gcc-c++

# Maven
RUN yum -y install maven

# SBT
RUN curl -Ls https://www.scala-sbt.org/sbt-rpm.repo > sbt-rpm.repo; \
	mv sbt-rpm.repo /etc/yum.repos.d/; 	yum -y install sbt

# Create the `build` user (for the development activities)
RUN adduser build
RUN echo "build ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/build && \
    chmod 0440 /etc/sudoers.d/build

# Set up the environment for the `build` user
RUN mkdir -p $BOM4V_DIR $HOME/.ssh && chmod 700 $HOME/.ssh
ADD resources/bashrc $HOME/.bashrc
RUN chmod 640 $HOME/.bashrc
ADD resources/Rakefile $BOM4V_DIR/
ADD resources/metamodels.yaml.sample $BOM4V_DIR/metamodels.yaml
RUN chown -R build.build $HOME

# Switch to the `build` user
USER build
WORKDIR $BOM4V_DIR

# Ruby - https://www.linkedin.com/pulse/how-install-ruby-centos-7-amal-krishna-r
RUN git clone https://github.com/rbenv/rbenv.git $HOME/.rbenv
RUN git clone https://github.com/rbenv/ruby-build.git $HOME/.rbenv/plugins/ruby-build

# Install Ruby
RUN source $HOME/.bashrc && \
	rbenv install $RUBY_VERSION && \
	rbenv global $RUBY_VERSION

# Install the nokogiri Gem
RUN source $HOME/.bashrc && gem install nokogiri --platform=ruby

# Prepare the build environment for the BOM4V project
WORKDIR $BOM4V_DIR

# Build the BOM for Verticals (BOM4V) components
RUN source $HOME/.bashrc && \
	rm -rf workspace && \
	rake clone && rake checkout && \
	cd workspace/src/ti-spark-data-generation && \
	./fillLocalDataDir.sh && cd $BOM4V_DIR && \
	cd workspace/src/ti-spark-examples && \
	./fillLocalDataDir.sh && cd $BOM4V_DIR && \
	rake offline=true deliver

# Entry point
CMD ["/bin/bash"]


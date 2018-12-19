#
# http://github.com/bom4v/metamodels/tree/master/docker/base
#
# Scala and SBT on CentOS 7 with Oracle Java JDK 8
# Inspired from: http://github.com/ecompositor/centos-scala
# Script to download Oracle JDK: https://gist.github.com/n0ts/40dd9bd45578556f93e7

FROM docker.io/centos:7
MAINTAINER Denis Arnaud <denis.arnaud_github at m4x dot org>

# Docker build time environment variables
ENV container docker
ENV HOME /home/build
ENV LANGUAGE en_US:en
ENV LANG en_US.UTF-8
ENV LC_ALL $LANG
ENV BOM4V_DIR $HOME/dev/bom4v

# Import the Centos-7 GPG key to prevent warnings
RUN rpm --import http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7

# Update of CentOS
RUN yum -y clean all && \
    yum -y upgrade && \
    yum -y install epel-release

# A few useful packages
RUN yum -y install less htop net-tools which sudo man vim \
	git-all wget curl file bash-completion keyutils \
	gzip tar unzip maven rake rubygem-rake rubygem-nokogiri

# Add bintray repository where the SBT binaries are published
RUN curl -sS https://bintray.com/sbt/rpm/rpm | \
	tee /etc/yum.repos.d/bintray-sbt-rpm.repo
RUN yum -y install sbt

# Java
ENV JDK_VERSION 8u192
ENV JDK_VERSION_MINOR b12
ENV JDK_RPM jdk-$JDK_VERSION-linux-x64.rpm
ENV JDK_RPM_TOKEN 750e1c8617c5452694857ad95c3ee230
ENV JDK_BASE_URL http://download.oracle.com/otn-pub/java/jdk
ENV JDK_RPM_URL $JDK_BASE_URL/$JDK_VERSION-$JDK_VERSION_MINOR/$JDK_RPM_TOKEN
ENV JDK_GETTER get_oracle_jdk_linux_x64.sh

# Scala
ENV SCALA_TAR_URL http://www.scala-lang.org/files/archive
ENV SCALA_VERSION 2.11.8
ENV SBT_VERSION 1.2.7
ENV SBT_OPTS -Xmx4G
ENV SBT_GBL $HOME/.sbt/1.2/global.sbt

# Download Java in a robust way
#ADD $JDK_GETTER /usr/local/bin
#RUN chmod a+x /usr/local/bin/$JDK_GETTER
#RUN /usr/local/bin/$JDK_GETTER

# Download Java in a ad hoc way (see https://stackoverflow.com/questions/10268583/downloading-java-jdk-on-linux-via-wget-is-shown-license-page-instead)
# and install it
RUN wget -c --header "Cookie: oraclelicense=accept-securebackup-cookie" $JDK_RPM_URL/$JDK_RPM && \
	yum -y install $JDK_RPM && rm -f $JDK_RPM

# Download and install Scala
RUN wget $SCALA_TAR_URL/scala-$SCALA_VERSION.tgz && \
	tar xvf scala-$SCALA_VERSION.tgz && \
	mv scala-$SCALA_VERSION /usr/lib && \
	ln -s /usr/lib/scala-$SCALA_VERSION /usr/lib/scala && \
	rm -f scala-$SCALA_VERSION.tgz

ENV PATH $PATH:/usr/lib/scala/bin

# Create the `build` user (for the development activities)
RUN adduser build
RUN echo "build ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/build && \
    chmod 0440 /etc/sudoers.d/build

# Configure SSH
RUN mkdir -p $HOME/.ssh && chmod 700 $HOME/.ssh
RUN ssh-keyscan github.com > $HOME/.ssh/known_hosts
ADD resources/ssh-config $HOME/.ssh/config
RUN chmod 600 $HOME/.ssh/config $HOME/.ssh/known_hosts

# Set up the packaging environment for the `build` user
ADD resources/bashrc $HOME/.bashrc
ADD resources/gitconfig $HOME/.gitconfig
ADD resources/vimrc $HOME/.vimrc
RUN chmod 640 $HOME/.bashrc $HOME/.gitconfig $HOME/.vimrc
RUN chown -R build.build $HOME

# Switch to the `build` user
WORKDIR $HOME
USER build

# Run SBT once, at Docker build time, so that it does not download
# the dependencies at run time
RUN mkdir -p $BOM4V_DIR
WORKDIR $BOM4V_DIR
RUN PATH=$PATH:/usr/lib/scala/bin sbt about

# Entry point
CMD ["/bin/bash"]

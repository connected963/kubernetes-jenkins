FROM jenkins/jenkins-experimental:latest-jdk11

EXPOSE 8080 50000

USER root

# Install prerequisites for Docker
RUN apt-get update && apt-get install -y sudo apt-transport-https ca-certificates maven iptables init-system-helpers libapparmor1 libltdl7 libseccomp2 libdevmapper1.02.1 curl  && rm -rf /var/lib/apt/lists/*

RUN curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
RUN echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | tee -a /etc/apt/sources.list.d/kubernetes.list

RUN apt-get update && apt-get install -y kubectl

RUN curl -sSL https://get.docker.com | sh

RUN curl -sOL http://updates.jenkins-ci.org/latest/jenkins.war
RUN rm /usr/share/jenkins/jenkins.war && mv jenkins.war /usr/share/jenkins/

# Repositories plugins
RUN /usr/local/bin/install-plugins.sh github
RUN /usr/local/bin/install-plugins.sh gitlab-plugin

# Distributed Builds plugins
RUN /usr/local/bin/install-plugins.sh ssh-slaves

# Install Notifications and Publishing plugins
RUN /usr/local/bin/install-plugins.sh email-ext
RUN /usr/local/bin/install-plugins.sh mailer
RUN /usr/local/bin/install-plugins.sh slack

# Artifacts
RUN /usr/local/bin/install-plugins.sh htmlpublisher

# UI
RUN /usr/local/bin/install-plugins.sh greenballs
RUN /usr/local/bin/install-plugins.sh simple-theme-plugin

# Scaling
RUN /usr/local/bin/install-plugins.sh kubernetes

ENTRYPOINT ["/sbin/tini", "--", "/usr/local/bin/jenkins.sh"]

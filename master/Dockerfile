FROM jenkins/jenkins:lts

COPY plugins.txt /usr/share/jenkins/ref/plugins.txt

ENV JENKINS_HOME /var/jenkins_home

RUN xargs /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt
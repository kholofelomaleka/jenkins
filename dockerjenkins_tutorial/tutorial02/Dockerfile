FROM jenkins/jenkins:lts
LABEL maintainer="kholofelo.maleka@yahoo.com"

ARG JAVA_OPTS="-Xmx256m" 

USER root
RUN mkdir /var/log/jenkins
RUN mkdir /var/cache/jenkins
RUN chown -R  jenkins:jenkins /var/log/jenkins
RUN chown -R jenkins:jenkins /var/cache/jenkins
USER jenkins

ENV JAVA_OPTS=$JAVA_OPTS

ENV JENKINS_OPTS="--logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war"

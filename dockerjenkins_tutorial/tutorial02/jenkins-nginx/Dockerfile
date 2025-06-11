FROM centos:centos7
LABEL maintainer="kholofelo.maleka@yahoo.com"

RUN rm -f /etc/yum.repos.d/*.repo
COPY repo/repos/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo
COPY repo/repos/CentOS-CR.repo /etc/yum.repos.d/CentOS-CR.repo
COPY repo/repos/CentOS-Debuginfo.repo /etc/yum.repos.d/CentOS-Debuginfo.repo
COPY repo/repos/CentOS-fasttrack.repo /etc/yum.repos.d/CentOS-fasttrack.repo
COPY repo/repos/CentOS-Media.repo /etc/yum.repos.d/CentOS-Media.repo
COPY repo/repos/CentOS-Sources.repo /etc/yum.repos.d/CentOS-Sources.repo
COPY repo/repos/CentOS-Vault.repo /etc/yum.repos.d/CentOS-Vault.repo
COPY repo/repos/CentOS-x86_64-kernel.repo /etc/yum.repos.d/CentOS-x86_64-kernel.repo

RUN yum repolist all
RUN yum clean all; yum -y update; yum clean all
RUN yum -y install http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm; yum -y makecache
RUN yum -y install nginx-1.10.3
RUN rm /etc/nginx/conf.d/default.conf

COPY conf/jenkins.conf /etc/nginx/conf.d/jenkins.conf
COPY conf/nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
CMD ["nginx"]

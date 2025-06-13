FROM centos:centos7
LABEL maintainer="kholofelo.maleka@yahoo.com"

RUN rm -f /etc/yum.repos.d/*.repo
COPY repo/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo
COPY repo/CentOS-CR.repo /etc/yum.repos.d/CentOS-CR.repo
COPY repo/CentOS-Debuginfo.repo /etc/yum.repos.d/CentOS-Debuginfo.repo
COPY repo/CentOS-fasttrack.repo /etc/yum.repos.d/CentOS-fasttrack.repo
COPY repo/CentOS-Media.repo /etc/yum.repos.d/CentOS-Media.repo
COPY repo/CentOS-Sources.repo /etc/yum.repos.d/CentOS-Sources.repo
COPY repo/CentOS-Vault.repo /etc/yum.repos.d/CentOS-Vault.repo
COPY repo/CentOS-x86_64-kernel.repo /etc/yum.repos.d/CentOS-x86_64-kernel.repo

# Install NGINX
RUN yum -y update; yum clean all
RUN yum -y install http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm; yum -y makecache
RUN yum -y install nginx-1.10.3

# Remove default files we don't need
RUN rm /etc/nginx/conf.d/default.conf

# Add default configuration
COPY conf/jenkins.conf /etc/nginx/conf.d/jenkins.conf
COPY conf/nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx"]

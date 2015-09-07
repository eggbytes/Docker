#   Leon Justice <leonljustice@eggbytes.com>
#   https://github.com/eggbytes/Nagios
#
#   CentOS 7, Nagios, NRPE, HTTPD, SSH, Supervisord, and Server Management Tools
#
#########################################################
#
#   The user accounts listed below are what you will use to administer this container. 
#   Please change the passwords for security needs.
#
#   Nagios Web Administration Account: uname: nagiosadmin passwd: nagiosadmin 
#   Shell, SSH, & Sudo Account: uname: nagiosadmin passwd: nagiosadmin
#   Nagios Service Account: uname: nagios passwd: nagios123
#   NRPE Service Account: uname: nrpe passwd: nrpe123
#   Nagios Service Group: nagcmd
#   
#    
#   Container Run Command Example 
#   "docker run -d -p 44333:22 -p 8080:80 -p 8993:443 -p 5666:5666 -p 12489:12489 leonljustice/nagios:latest"
#

FROM centos:latest
MAINTAINER The CentOS Project <cloud-ops@centos.org>


# YUM PACKAGE INSTALLATION
RUN yum -y install epel-release
RUN yum -y update; yum clean all
RUN rpm -e vim-minimal-7.4.160-1.el7.x86_64
RUN yum -y install vim; yum -y clean all

RUN yum -y group install "Development Tools"; yum clean all

Run yum -y install openssh-server \
    passwd \
    sudo \
    bind-utils \
    which \
    wget \
    git \
    net-tools \
    python-psutil \
    openssl \
    openssl-devel \
    ssh \
    xinetd \
    apache2-utils \
    httpd \
    php \
    gcc \
    glibc \
    glibc-common \
    gd \
    gd-devel \
    python-setuptools; yum -y clean all


RUN easy_install supervisor

#################################################

# ACCOUNT CREATION FOR RUNNING NAGIOS, AND NRPE
RUN useradd -m nagios
RUN echo "nagios:nagios123" | chpasswd
RUN useradd -m nrpe
RUN echo "nrpe:nrpe123" | chpasswd
RUN useradd nagiosadmin


# CREATING CONTAINER SHELL, SSH AND SUDO ACCOUNT FOR MANAGEMENT
RUN echo "nagiosadmin:nagiosadmin" | chpasswd
RUN echo 'nagiosadmin  ALL=(ALL:ALL) ALL' >> /etc/sudoers


# CREATING NAGCMD GROUP THEN ADDING NAGIOS, NRPE, & APACHE USERS
RUN groupadd nagcmd
RUN usermod -a -G nagcmd nagios
RUN usermod -a -G nagcmd nrpe
RUN usermod -a -G nagcmd apache


# NAGIOS, NAGIOS PLUGINS, & NRPE DEPLOYMENT
RUN wget http://prdownloads.sourceforge.net/sourceforge/nagios/nagios-4.0.8.tar.gz
RUN wget http://nagios-plugins.org/download/nagios-plugins-2.0.3.tar.gz
Run wget http://sourceforge.net/projects/nagios/files/nrpe-2.x/nrpe-2.15/nrpe-2.15.tar.gz


# NAGIOS CORE 4.0.8 DEPLOYMENT
RUN  tar -xzvf nagios-4.0.8.tar.gz
RUN cd nagios-4.0.8; ./configure --with-command-group=nagcmd; make all; make install; make install-init; make install-config; make install-commandmode; make install-webconf


# NAGIOS PLUGIN 2.0.3 DEPLOYMENT
RUN cd ..
RUN tar -xzvf nagios-plugins-2.0.3.tar.gz
RUN cd nagios-plugins-2.0.3; ./configure --with-nagios-user=nagios --with-nagios-group=nagios; make && make install


# NRPE 2.15 DEPLOYMENT
RUN cd ..
RUN tar -xzvf nrpe-2.15.tar.gz
RUN cd nrpe-2.15; ./configure --enable-command-args --with-ssl=/usr/bin/openssl --with-ssl-lib=/usr/lib/x86_64-linux-gnu --with-nrpe-user=nrpe --with-nrpe-group=nagcmd --with-nagios-user=nagios --with-nagios-group=nagcmd --libexecdir=/usr/local/nagios/libexec/ --bindir=/usr/local/nagios/bin/ --prefix=/usr/local/nagios; make all; make install; make install-xinetd; make install-daemon-config


# REMOVING NAGIOS INSTALLATION PACKAGES
RUN cd ..
RUN rm -rf nagios-4.0.8*
RUN rm -rf nrpe-2.15*
RUN rm -rf nagios-plugins-2.0.3*


# REMOVING PACKAGES THAT WERE USED TO BUILD NAGIOS
Run yum -y group remove "Development Tools"; yum clean all


# CREATING NAGIOS WEB ADMINISTRATION ACCOUNT
RUN htpasswd -b -c /usr/local/nagios/etc/htpasswd.users nagiosadmin nagiosadmin


# SSH KEY CREATION
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key -N ''


# CREATING DIRECTORIES
RUN mkdir -p /etc/supervisor/conf.d \
             /var/log/supervisor \
             /var/run/sshd \
             /var/log/sshd \
             /var/log/apache2 \
             /var/run/apache2


# SETTING ENVIROMENT VARIABLES
ENV TERM xterm
ENV SHELL /bin/bash
ENV PATH /usr/local/bin:/usr/bin:/bin:/usr/sbin:/usr/local/nagios/bin:/usr/local/nagios/libexec:/usr/sbin:/sbin:/


# ADDING SUPERVISORD CONFIGURATION FILES
ADD sshd.ini /etc/supervisor/conf.d/sshd.ini
ADD apache2.ini /etc/supervisor/conf.d/apache2.ini
ADD nagios.ini /etc/supervisor/conf.d/nagios.ini
ADD supervisord.conf /etc/supervisord.conf


# CLEANING MY ROOM
RUN yum -y clean all; yum -y check
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*


# OPEN NEEDED PORTS FOR NAGIOS
EXPOSE 22 80 443 5666 12489


# COMMAND TO LAUNCH SUPERVISORD
CMD /usr/bin/supervisord  -c /etc/supervisord.conf

#END

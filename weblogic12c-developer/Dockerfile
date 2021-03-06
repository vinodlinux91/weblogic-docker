# LICENSE CDDL 1.0 + GPL 2.0
#
# ORACLE DOCKERFILES PROJECT
# --------------------------
# This is the Dockerfile for WebLogic 12.1.3 ZIP Distro
# 
# REQUIRED BASE IMAGE TO BUILD THIS IMAGE
# ---------------------------------------
# Make sure you have oraclelinux:7.0 Docker image installed.
# Visit for more info: http://public-yum.oracle.com/docker-images/
#
# REQUIRED FILES TO BUILD THIS IMAGE
# ----------------------------------
# (1) wls1213_dev.zip
#     Download from http://www.oracle.com/technetwork/middleware/weblogic/downloads/index.html
#
# (2) jdk-8u25-linux-x64.rpm
#     Download from http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
#
# HOW TO BUILD THIS IMAGE
# -----------------------
# Put all downloaded files in the same directory as this Dockerfile
# Run: 
#      $ sudo docker build -t oracle/weblogic:12.1.3 .
#
# DOCKER SCRIPTS FOR BUILDING/RUNNING WEBLOGIC
# --------------------------------------------
# You can also use the scripts provided in the folder ../bin to build and run WebLogic on Docker. 
# Example building:
#          $ sudo buildDockerImage.sh [-d]
#                -d: builds the developer image
#
# Example running: 
#          $ sudo dockWebLogic.sh [-d] 
#                 -d: runs the developer image
#

# Pull base image
# ---------------
FROM oraclelinux:7.0

# Maintainer
# ----------
MAINTAINER Bruno Borges <bruno.borges@oracle.com>

# Environment variables required for this build (do NOT change)
ENV JAVA_RPM jdk-8u25-linux-x64.rpm
ENV WLS_PKG  wls1213_dev.zip

# WLS Admin Password (you may change)
# This password is used for:
#  (a) 'weblogic' admin user of WebLogic 
#  (b) 'oracle' Linux user in this image
# -----------------------------------
ENV ADMIN_PASSWORD welcome1

# Install and configure Oracle JDK 8u25
# -------------------------------------
ADD $JAVA_RPM /root/
RUN rpm -i /root/$JAVA_RPM && \ 
    rm /root/$JAVA_RPM
ENV JAVA_HOME /usr/java/default
ENV MW_HOME /u01/oracle/wls12130
ENV CONFIG_JVM_ARGS -Djava.security.egd=file:/dev/./urandom

# Setup filesystem and oracle user
# ------------------------------------------------------------
# Enable this if behind proxy
RUN mkdir /u01 && chmod a+xr /u01 && \ 
    useradd -b /u01 -m -s /bin/bash oracle && \ 
    echo oracle:$ADMIN_PASSWORD | chpasswd

# Add files required to build this image
ADD $WLS_PKG /u01/
ADD container-scripts/* /u01/
RUN mv /u01/add-machine.py /u01/createMachine.sh /u01/oracle/

# Adjust file permissions, go to /u01 as user 'oracle' to proceed with WLS installation
RUN chown oracle:oracle -R /u01
WORKDIR /u01
USER oracle

# Installation of Weblogic 
WORKDIR /u01/oracle/
RUN jar xf /u01/$WLS_PKG
RUN ln -s /u01/oracle/wls12130 /u01/oracle/weblogic
RUN chown oracle:oracle -R /u01

WORKDIR /u01/oracle/weblogic
RUN sh configure.sh -silent
RUN find /u01/oracle/wls12130 -name "*.sh" -exec chmod a+x {} \;
RUN /u01/oracle/weblogic/wlserver/common/bin/wlst.sh -skipWLSModuleScanning /u01/create-wls-domain.py && \ 
    mkdir -p /u01/oracle/weblogic/user_projects/domains/base_domain/servers/AdminServer/security && \ 
    mv /u01/commEnv.sh /u01/oracle/weblogic/wlserver/common/bin/commEnv.sh && \ 
    echo "username=weblogic" > /u01/oracle/weblogic/user_projects/domains/base_domain/servers/AdminServer/security/boot.properties && \ 
    echo "password=$ADMIN_PASSWORD" >> /u01/oracle/weblogic/user_projects/domains/base_domain/servers/AdminServer/security/boot.properties && \ 
    rm /u01/create-wls-domain.py /u01/$WLS_PKG /u01/jaxrs2-template.jar && \ 
    echo ". /u01/oracle/weblogic/user_projects/domains/base_domain/bin/setDomainEnv.sh" >> /u01/oracle/.bashrc && \ 
    echo "export PATH=$PATH:/u01/oracle/weblogic/wlserver/common/bin:/u01/oracle/weblogic/user_projects/domains/base_domain/bin" >> /u01/oracle/.bashrc

# Expose Node Manager default port, and also default http/https ports for admin console
EXPOSE 5556 7001 7002

# Make sure root has also right PATH and WLS environment variables configured
USER root
RUN echo ". /u01/oracle/weblogic/user_projects/domains/base_domain/bin/setDomainEnv.sh" >> /root/.bashrc && \ 
    echo "export PATH=$PATH:/u01/oracle/weblogic/wlserver/common/bin:/u01/oracle/weblogic/user_projects/domains/base_domain/bin" >> /root/.bashrc

WORKDIR /u01/oracle
USER oracle

# Define default command to start bash. 
CMD ["/u01/oracle/weblogic/user_projects/domains/base_domain/bin/startWebLogic.sh"]

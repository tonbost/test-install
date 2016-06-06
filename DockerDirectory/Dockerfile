############################################################
# Dockerfile to run oracle 12c
# Based on Centos7 Image
############################################################

# Set the base image to use to Centos
FROM centos
# Set the file maintainer (your name - the file's author)
MAINTAINER zdz
# Update the default application repository sources list
RUN yum update

COPY oracle-12c/step1/linuxamd64_12102_database_1of2.zip /tmp/install/linuxamd64_12102_database_1of2.zip
COPY oracle-12c/step1/linuxamd64_12102_database_2of2.zip /tmp/install/linuxamd64_12102_database_2of2.zip

# RUN yum -y install zip unzip binutils compat-libcap1 compat-libstdc++-33 compat-libstdc++-33.i686 gcc gcc-c++ glibc.i686 glibc glibc-devel glibc-devel.i686 ksh libgcc.i686 libgcc libstdc++ libstdc++.i686 libstdc++-devel libstdc++-devel.i686 libaio libaio.i686 libaio-devel libaio-devel.i686 libXext libXext.i686 libXtst libXtst.i686 libX11 libX11.i686 libXau libXau.i686 libxcb libxcb.i686 libXi libXi.i686 make sysstat vte3 smartmontools
RUN yum install -y zip unzip binutils.x86_64 compat-libcap1.x86_64 gcc.x86_64 gcc-c++.x86_64 glibc.i686 glibc.x86_64 \
glibc-devel.i686 glibc-devel.x86_64 ksh compat-libstdc++-33 libaio.i686 libaio.x86_64 libaio-devel.i686 libaio-devel.x86_64 \
libgcc.i686 libgcc.x86_64 libstdc++.i686 libstdc++.x86_64 libstdc++-devel.i686 libstdc++-devel.x86_64 libXi.i686 libXi.x86_64 \
libXtst.i686 libXtst.x86_64 make.x86_64 sysstat.x86_64

RUN cd /tmp/install && unzip linuxamd64_12102_database_1of2.zip && unzip linuxamd64_12102_database_2of2.zip && rm *.zip

RUN groupadd -g 54321 oinstall && groupadd -g 54322 dba
# RUN userdel oracle && rm -rf /home/oracle && rm /var/spool/mail/oracle
RUN useradd -m -u 54321 -g oinstall -G dba oracle
RUN echo "oracle:oracle" | chpasswd

ENV ORACLE_BASE /u01/app/oracle
RUN mkdir -p $ORACLE_BASE && chown -R oracle:oinstall $ORACLE_BASE && chmod -R 775 $ORACLE_BASE
RUN mkdir -p /u01/app/oraInventory && chown -R oracle:oinstall /u01/app/oraInventory && chmod -R 775 /u01/app/oraInventory

COPY oracle-12c/step1/oraInst.loc /etc/oraInst.loc
RUN chmod 664 /etc/oraInst.loc
COPY oracle-12c/step1/sysctl.conf /etc/sysctl.conf
# RUN echo "oracle soft stack 10240" >> /etc/security/limits.conf
RUN echo "oracle soft nproc 2047" >> /etc/security/limits.conf
RUN echo "oracle hard nproc 16384" >> /etc/security/limits.conf
RUN echo "oracle soft nofile 1024" >> /etc/security/limits.conf
RUN echo "oracle hard nofile 65536" >> /etc/security/limits.conf

ENV CVUQDISK_GRP oinstall
RUN cd /tmp/install/database/rpm && rpm -iv cvuqdisk-1.0.9-1.rpm
COPY oracle-12c/step1/db_install.rsp /tmp/db_install.rsp
COPY oracle-12c/step1/install /tmp/install/install

RUN yum -y clean all

# Install DB
RUN /tmp/install/install

# Step2
RUN rm -rf /tmp/install

ENV ORACLE_SID ORCL
ENV ORACLE_HOME $ORACLE_BASE/product/12.1.0/dbhome_1
ENV PATH $ORACLE_HOME/bin:$PATH

RUN $ORACLE_HOME/root.sh

RUN mkdir -p $ORACLE_BASE/oradata && chown -R oracle:oinstall $ORACLE_BASE/oradata && chmod -R 775 $ORACLE_BASE/oradata
RUN mkdir -p $ORACLE_BASE/fast_recovery_area && chown -R oracle:oinstall $ORACLE_BASE/fast_recovery_area && chmod -R 775 $ORACLE_BASE/fast_recovery_area

COPY oracle-12c/step2/initORCL.ora $ORACLE_HOME/dbs/initORCL.ora
COPY oracle-12c/step2/createdb.sql $ORACLE_HOME/config/scripts/createdb.sql
COPY oracle-12c/step2/conf_finish.sql $ORACLE_HOME/config/scripts/conf_finish.sql
COPY oracle-12c/step2/create /tmp/create
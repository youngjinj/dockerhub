FROM centos:7

MAINTAINER Joo Youngjin <youngjinj@cubrid.com>

RUN yum update -y \
    && yum install -y epel-release \
    && yum install -y centos-release-scl \
    && yum install -y sudo \
                      vim \
                      which \
                      net-tools \
                      telnet \
                      nc \
                      wget \
                      expect \
                      openssh-server \
                      openssh-clients \
                      sshpass \
    && yum install -y devtoolset-8 \
                      scl-utils \
                      java-1.8.0-openjdk-devel \
                      git \
                      cmake \
                      gcc-c++ \
                      systemtap \
                      systemtap-sdt-devel \
                      bison \
                      flex \
                      ncurses-devel \
                      ant \
                      elfutils-libelf-devel \
                      libtool \
                      rpm-build \
    && yum install -y gdb \
                      ctags \
                      cscope \
    && yum clean all

RUN localedef -f UTF-8 -i ko_KR ko_KR.utf8 \
    && ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime

ENV LC_ALL ko_KR.utf8
ENV LANG ko_KR.utf8
ENV LANGUAGE ko_KR.utf8

RUN ssh-keygen -f /etc/ssh/ssh_host_rsa_key -t rsa -N "" \
    && ssh-keygen -f /etc/ssh/ssh_host_ecdsa_key -t ecdsa -N "" \
    && ssh-keygen -f /etc/ssh/ssh_host_ed25519_key -t ed25519 -N "" \
    && chmod u+s /usr/bin/ping

RUN mkdir /docker-entrypoint-initdb.d

RUN useradd cubrid \
    && sed 102s/#\ %wheel/%wheel/g -i /etc/sudoers \
    && sed s/wheel:x:10:/wheel:x:10:cubrid/g -i /etc/group \
    && sed -e '61 i\cubrid\t\t soft\t nofile\t\t 65536 \
    cubrid\t\t hard\t nofile\t\t 65536 \
    cubrid\t\t soft\t core\t\t unlimited \
    cubrid\t\t hard\t core\t\t unlimited\n' -i /etc/security/limits.conf \
    && echo -e "\ncubrid     soft    nproc     65546\ncubrid     hard    nproc     65536" >> /etc/security/limits.d/20-nproc.conf

RUN mkdir -p /home/cubrid/github/cubrid \
    && git clone -b develop --recursive https://github.com/youngjinj/cubrid.git /home/cubrid/github/cubrid

RUN source scl_source enable devtoolset-8 \
    && cd /home/cubrid/github/cubrid \
    && ./build.sh -m debug -p /home/cubrid/CUBRID -j /usr/lib/jvm/java build \
    && echo 'java_stored_procedure=y' >> /home/cubrid/CUBRID/conf/cubrid.conf \
    && mkdir -p /home/cubrid/CUBRID/databases \
                /home/cubrid/CUBRID/tmp \
                /home/cubrid/CUBRID/var/CUBRID_SOCK

COPY cubrid.sh /home/cubrid/

RUN echo '' >> /home/cubrid/.bash_profile \
    && echo 'export LC_ALL=ko_KR.utf8' >> /home/cubrid/.bash_profile \
    && echo 'export LANG=ko_KR.utf8' >> /home/cubrid/.bash_profile \
    && echo 'export LANGUAGE=ko_KR.utf8' >> /home/cubrid/.bash_profile \
    && echo '' >> /home/cubrid/.bash_profile \
    && echo 'umask 077' >> /home/cubrid/.bash_profile \
    && echo '' >> /home/cubrid/.bash_profile \
    && echo '. /home/cubrid/cubrid.sh' >> /home/cubrid/.bash_profile \
    && echo '' >> /home/cubrid/.bash_profile \
    && echo 'source scl_source enable devtoolset-8' >> /home/cubrid/.bash_profile \
    && chown -R cubrid:cubrid /home/cubrid

COPY docker-entrypoint.sh /usr/local/bin

RUN chmod +x /usr/local/bin/docker-entrypoint.sh \
    && ln -sf /usr/local/bin/docker-entrypoint.sh /entrypoint.sh

EXPOSE 22 30000

ENTRYPOINT ["docker-entrypoint.sh"]
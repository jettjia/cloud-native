FROM jenkins/jnlp-slave:latest

MAINTAINER itcast

# 切换到 root 账户进行操作
USER root

# 安装 maven
COPY apache-maven-3.6.2-bin.tar.gz .

RUN tar -zxf apache-maven-3.6.2-bin.tar.gz && \
    mv apache-maven-3.6.2 /usr/local && \
    rm -f apache-maven-3.6.2-bin.tar.gz && \
    ln -s /usr/local/apache-maven-3.6.2/bin/mvn /usr/bin/mvn && \
    ln -s /usr/local/apache-maven-3.6.2 /usr/local/apache-maven && \
    mkdir -p /usr/local/apache-maven/repo

COPY settings.xml /usr/local/apache-maven/conf/settings.xml

USER jenkins
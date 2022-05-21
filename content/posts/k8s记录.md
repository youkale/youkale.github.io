---
title: k8s记录
date: 2019-02-18 15:55:36
tag:
---

# 用rancher配置k8s环境

## 删除已经安装的docker

    yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-selinux \
                docker-engine-selinux \
                docker-engine

    yum install -y yum-utils \
        device-mapper-persistent-data \
        lvm2

    yum-config-manager \
        --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo

    yum install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm

    yum install docker-ce-17.03.2.ce-1.el7.centos

## 配置 rancher

    docker pull rancher/server:v1.6.14

    docker run -d --name rancher-mysql -p 3306:3306 -v /opt/rancher/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=rancher_mysql -d percona:5.7.22-stretch --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

    docker run -it --rm --link  rancher-mysql

    create database rancher ;
    GRANT ALL ON *.* TO 'root'@'%' IDENTIFIED BY 'rancher_mysql';
    GRANT ALL ON rancher.* TO 'cattle'@'localhost' IDENTIFIED BY 'rancher';

    docker run -d --restart=unless-stopped -p 8080:8080 rancher/server:v1.6.14 --db-host 172.20.2.35 --db-port 3306 --db-user root --db-pass rancher_mysql

    docker run -d -p 3306:3306 -v /data/rancher/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=rancher_mysql --restart=unless-stopped -p 8080:8080 rancher/server:v1.6.21

    docker run -d  -v /opt/rancher/mysql:/var/lib/mysql  --restart=unless-stopped -p 8080:8080 rancher/server:v1.6.14

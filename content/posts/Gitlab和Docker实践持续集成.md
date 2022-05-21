---
title: Gitlab和Docker实践持续集成
date: 2018-08-29 10:45:37
tag:
---

# 前言

>本文面向那些每次发包,都是本地构建,通过奇慢无比的网络进行上传,一旦打包出错(or 没有fix bug)，还需要重新上传,无穷无尽的任务还在等着,而此刻网络成了瓶颈。又或者此刻测试人员正在测试,而你停了服务,造成测试工作无法进行,此刻的你已经是万人瞩目了,测试、项目经理已经磨刀霍霍了.就问你慌不慌?

## 打包

>上面说的情况其实是程序发版,说到发版目前各个系统程序交付形式有大概分为这些: windows常见的zip,rar,msi等; Linux常见的rpm,deb等; mac常见的dmg等;java语言的有jar,war包等; 然而多种的打包形式,依赖的环境不一样,每种包也只能是在特定的环境下才能使用,这其实对运维也是一种挑战. 那么有没有一种包是各个平台都可以用，而且不用去关心目标机器环境的呢? 答案就是: docker,下面我们就结合docker,进行自动化打包, 缩短各个职能间沟通的成本.

## Docker

>玩过docker的人都知道,这玩意真心好,都不用为环境操心了, 运维也只需要关心namespace和cgroups就好了，不用在去管什么rpm包,deb包,tar.gz扽等了。

## 使用工具

    Gitlab: 代码仓库, 构建工具(pipeline)
    Nexus:  maven仓库, docker register (新版Gitlab已经包含Registery)
    Docker: 容器
    Webhook: 自己写的 [Github] (https://github.com/youkale/gitlab-webhook)

## 基本流程

                                                    3
                                       +-----------------------------+
                                       |                             |
                                       |                             |
            +---------+   1     +------v-------+            +--------+------+
            |         |         |              |     2      |               |
            |  Coder  +--------->    Gitlab    +------------> Gitlab Runner |
            |         |         |              |            |               |
            +---------+         +---^----+-----+            +--------+------+
                                    |    |                           |
                                    |    |                           |
                                    |    |  +-----------------+      | 4
                                    |    |  |                 |      |
                                    |    |  | Docker Registry <------+
                                    |    |  |                 |
                                    |   5|  +-------^---------+
                                   7|    |          |
                                    |    |          | 6
                                    |    |  +-------+---------+
                                    |    |  |                 |
                                    |    +--> Target Server   |
                                    |       |                 |
                                    |       +-------+---------+
                                    |               |
                                    +---------------+

    1. 程序员push代码到gitlab，(1)
    2. Gitlab将任务分配给GitRunner，可以根据配置将执行结果(jar,js等)upload到gitlab或是打包docker镜像push到docker仓库. 也可以根据分支,tag指定构建 (2,3,4)
    3. 执行完毕后会将发送一个构建事件到你注册的Webhook里. (5)
    4. 目标服务器会装有webhook根据Gitlab事件信息拉取docker镜像,也可以从Gitlab拉去config文件或者其他的Artifacts. (6,7)
> 大致的流程就是这样,Gitlab可以将各个环节需要用的Token，通过环境变量的形式添加到运行配置里面(.gitlab-ci.yml),从而避免了密钥泄露的情况.看完里这个，也许大家会在想，这个其实跟我们常用的jenkins差不多嘛，其实我安利gitlab的原因是它的一系列访问api、pipeline、webhook、artifacts、docker支持、用户友好、k8s的支持也很好.

## 准备

    构建前需要准备Docker基础镜像,比如java的maven依赖镜像,golang的GOPATH镜像,js的node基础镜像
    下面java工程我用的就是maven:3.5.2-jdk-8-alpine这个镜像

## 构建java工程

    image: docker.youkale.com:18443/docker:17.12.1-ce
    services:
      - docker.youkale.com:18443/docker:dind
    stages:
      - build
      - package
    variables:
      api_version: 1.0.8-api-release
    api-build:
      image: docker.youkale.com:18443/maven:3.5.2-jdk-8-alpine
      stage: build
      script:
        - ./mvnw clean package spring-boot:repackage -DskipTests=true
      artifacts:
        name: "$CI_JOB_NAME"
        expire_in: 1 weeks
        paths:
          - target/api.jar
    api-package:
      stage: package
      script:
      - "docker build -t docker.youkale.com:18442/api:${api_version} ./api"
      - "docker login -u gitlab-ci -p ${nexus_docker_password} docker.youkale.com:18442"
      - "docker push docker.youkale.com:18442/api:${api_version}"

    说明:
        image: 执行本次构建使用的image
        stages: 执行本次构建的顺序, build,package
        variables: 环境变量
        api-build: 自定义构建名称,会定义构建的策略.上传到artifacts会以这个命名.同样的webhook收到的事件也会包含此构建的名字,以及其他git相关的信息.我们根据这个信息进行应用的部署.
        ${nexus_docker_password}: 通过gitlab在运行的时候设置

## 其他

    webhook配置: https://raw.githubusercontent.com/youkale/gitlab-webhook/master/README.md
    nexus docker仓库配置: 难度倒是没有，需要(hosted,proxy,group), 但是还是走了不少弯路，比如docker login有问题等,不过最后也还是解决了.
    gitlab-runner: Settings -> CI/CD -> Runners
    gitlab-webhook: Settings -> Integrations
    分支、tag保护: Settings -> Repository

>gitlab构建部分的功能基本说说完了,每个地方还有不少细节,当我们实际用到的时候，我们可以结合gitlab详细的文档进行参考配置.

## 坑

    1. private-token问题:
    当使用官方gitlab的时候private-token在头部，而且全部大写，当使用gitlab-ce版的时候，private-token为首字母大写Private-Token(没记错的话)
    例子: curl -XGET -L --header \"PRIVATE-TOKEN: ${gitlab_artifact_token}\" 'https://gitlab.com/youkale/kakaka/-/jobs/artifacts/master/raw/dist/index.html?job=node-build' -o src/main/resources/templates/index.html
    2. 用这种模式构建，java工程的话不太推荐父子工程(本人主要用java)，当两个子工程都需要构建的时候，会不太舒服，虽然说这个方案也行

## 最后

>通过这种方式，我想开发乐呵呵，测试乐呵呵，运维乐呵呵. 大家都不用那么痛苦了.而且可玩的方式也很多，不一定需要k8s，对于小于20个docker的项目我觉得还可以结合docker-compose来部署.也不需要每一个新环境花费一天半天的时间来搭建.

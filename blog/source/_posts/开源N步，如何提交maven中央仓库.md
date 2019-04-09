---
title: 开源N步，如何提交maven中央仓库
date: 2019-04-09 09:30:54
tags:
---
Apache Maven 是源码管理和工程构建工具，我们也习惯了如何从中央仓库中通过groupId和artifactId去管理自己的项目依赖。

在公司，我们可能也习惯使用公司私有的maven仓库去发布我们的源码包，但是如何去把我们的开源项目包提交中央仓库以提供更多使用者呢？

首先，需要了解中央仓库是什么组织在管理的

[sonatype](https://www.sonatype.com/)

[sonatype document](https://help.sonatype.com/docs)

[sonatype jira](https://issues.sonatype.org)

所以，我们要提交开源包中央仓库就需要sonatype的账户

去[sonatype项目管理工具](https://issues.sonatype.org)中申请账户，然后创建一个自己的项目

```
Project 选择Community Support

Issue Type 选择New Project

Summary 填上项目名称

Description 填写项目描述

Group Id 填写项目的groupId，规则需要按照 [中央仓库GroupId规则](http://central.sonatype.org/pages/choosing-your-coordinates.html)

Project URL 填写项目的地址

SCM url 填写项目的源码管理地址，如github管理的就填写github项目地址

```

以上是一个新Project必要的字段，如没有，则需要Configure Fields中勾选相应的字段。

提交新项目申请后会有负责人审核项目，审核过程会用邮件告知

审核成功后该 Issue 状态会变更成 RESOLVED 状态，Comments中也会提醒你如何去处理。

接下来就是如同发布到私有库中的流程一样了，只不过中央库更加严格

```
首先pom中需要声明项目地址如：<url>https://github.com/mosence/xxx</url>

也需要声明开源协议如Apache开源协议
<licenses>
    <license>
        <name>The Apache License, Version 2.0</name>
        <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
    </license>
</licenses>

开发者信息
<developers>
    <developer>
        <name>mosence</name>
        <email>mosence@gmail.com</email>
    </developer>
</developers>

项目源码管理地址
<scm>
    <connection>scm:git:https://github.com/mosence/xxx.git</connection>
    <developerConnection>scm:git:https://github.com/mosence/xxx.git</developerConnection>
    <url>https://github.com/mosence/xxx</url>
    <tag>v0.0.1</tag>
</scm>

发布管理中依照Comments中的提醒去添加发布地址
<distributionManagement>
    <snapshotRepository>
        <id>sonatype</id>
        <url>https://oss.sonatype.org/content/repositories/snapshots</url>
    </snapshotRepository>
    <repository>
        <id>sonatype</id>
        <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
    </repository>
</distributionManagement>

插件中需要添加maven-release-plugin、maven-source-plugin、maven-javadoc-plugin来进行发布、源码发布、javadoc发布

nexus-staging-maven-plugin来配置提交规范

maven-gpg-plugin来对发布包进行gpg签名

相应的插件用法就不详细介绍了

```

然后通过 mvn deploy 去发布你的开源包

最后提醒一下版本号，-SNAPSHOT后缀会发布到快照版本库中，其他后缀发布在正式版本库中，正式版本库是没法覆盖删除的，请注意！
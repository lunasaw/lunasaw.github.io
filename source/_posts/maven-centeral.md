---
title: maven to centeral
date: 2021-05-11 16:35:41
banner_img: /img/basic/maven-binner.jpg
index_img: /img/basic/maven-index.jpg
tags: 
 - maven
categories:
 - basic-component
 - maven
---

# [如何将JAR包发布到Maven中央仓库？]()

将jar包发布到Maven中央仓库([Maven Central Repository](https://mvnrepository.com/))，这样所有的Java开发者都可以使用

Maven直接导入依赖,例如

```xml
<!-- https://mvnrepository.com/artifact/com.github.lunasaw/luna-common -->
<dependency>
    <groupId>com.github.lunasaw</groupId>
    <artifactId>luna-common</artifactId>
    <version>1.1.0</version>
</dependency>
```

但是，Maven中央仓库并不支持直接发布jar包。我们需要将jar包发布到一些指定的第三方Maven仓库，然后该仓库再将jar包同步到Maven中央仓库。

其中，最"简单"的方式是通过[Sonatype OSSRH](https://central.sonatype.org/pages/ossrh-guide.html)仓库来发布jar包。接下来，我会介绍如何将jar包发布到Sonatype OSSRH。

本教程所使用的系统配置如下：

- OS：macOS 10.14.2
- JDK：1.8.0_192
- Maven：3.5.4

### 1. 注册JIRA账号

JIRA是一个项目管理服务，类似于国内的Teambition。Sonatype通过JIRA来管理OSSRH仓库。

注册地址：https://issues.sonatype.org/secure/Signup!default.jspa

需要填写Email, Full Name, Username以及password，其中**Username与Password后面的步骤需要用到**，请记下来。

### 2. 创建issue

通过在JIRA上创建issue来申请发布新的jar包，Sonatype的工作人员会进行审核，审核不算严格，一般按照要求填写不会有问题。

创建链接：https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134

![img](https://i.loli.net/2021/05/11/AhUElLmIf5CjKqk.png)

创建issue的时候需要填写下面这些信息：

- Summary
- Description
- Group Id
- Project URL
- SCM url

大家可以参考我申请发布[fundebug-java](https://mvnrepository.com/artifact/com.fundebug/fundebug-java)与[fundebug-spring](https://mvnrepository.com/artifact/com.fundebug/fundebug-spring)时所填写的内容：[OSSRH-45238](https://issues.sonatype.org/browse/OSSRH-45238)

由于时差，前一天创建issue，第二天早上才会有回应。当issue的status变为**RESOLVED**，我们就可以进行下一步操作了。

### 3. 安装并配置GPG

发布到Maven仓库中的所有文件都要使用GPG签名，以保障完整性。因此，我们需要在本地安装并配置GPG。

**安装GPG**

MacBook安装GPG非常简单，下载并安装[GPG Suite](https://gpgtools.org/)即可。

**生成GPG密钥对**

```bash
gpg --gen-key
```

生成密钥时将需要输入name、email以及password。**password在之后的步骤需要用到**，请记下来。

**上传GPG公钥**

将公钥上传到公共的密钥服务器，这样其他人才可以通过公钥来验证jar包的完整性。

```bash
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys CAB4165C69B699D989D2A62BD74A11D3F9F41243
```

其中**CAB4165C69B699D989D2A62BD74A11D3F9F41243**为密钥的ID，可以通过**gpg --list-keys**命令查看

```bash
gpg --list-keys

/Users/kiwenlau/.gnupg/pubring.kbx
----------------------------------
pub   dsa2048 2010-08-19 [SC] [expires: 2020-06-15]
      85E38F69046B44C1EC9FB07B76D78F0500D026C4
uid           [ unknown] GPGTools Team <team@gpgtools.org>
sub   elg2048 2010-08-19 [E] [expires: 2020-06-15]
sub   rsa4096 2014-04-08 [S] [expires: 2024-01-02]

pub   rsa2048 2019-01-03 [SC] [expires: 2021-01-02]
      CAB4165C69B699D989D2A62BD74A11D3F9F41243
uid           [ultimate] kiwenlau <kiwenlau@gmail.com>
sub   rsa2048 2019-01-03 [E] [expires: 2021-01-02]
```

### 4. 配置Maven的setting.xml

[setting.xml](https://www.cnblogs.com/fundebug/p/how-to-deploy-jar-to-maven-central-repository.html)为Maven的全局配置文件，在MacBook上的位置为**/usr/local/Cellar/maven/3.5.4/libexec/conf/settings.xml**，我们需要将**第1步**配置的Username和Password添加到`<servers></servers>`标签中，这样我们才能将jar包部署到Sonatype OSSRH仓库：

```xml
<servers>
        <server>
                <id>ossrh</id>
                <username>Fundebug</username>
                <password>passsword</password>
        </server>
</servers>
```

### 5. 配置项目的pom.xml

**pom.xml**挺长的。根据Sonatype OSSRH的[要求](https://central.sonatype.org/pages/requirements.html)，以下信息都必须配置：

- Supply Javadoc and Sources
- Sign Files with GPG/PGP
- Sufficient Metadata
  - Correct Coordinates
  - Project Name, Description and URL
  - License Information
  - Developer Information
  - SCM Information

配置时参考我的pom.xml，根据需要修改即可。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.github.lunasaw</groupId>
    <artifactId>luna-common</artifactId>
    <name>luna-common</name>
    <version>1.1.1</version>
    <description>common is project which contains common utils</description>
    <url>https://github.com/lunasaw/luna-common</url>

    <properties>
        <java.version>1.8</java.version>
        <!-- github server corresponds to entry in ~/.m2/settings.xml -->
        <github.global.server>github_auth</github.global.server>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <!--dependencies-->
    </properties>
    <developers>
        <developer>
            <name>luna</name>
            <id>luna</id>
            <email>iszychen@gmail.com</email>
        </developer>
    </developers>
    <licenses>
        <license>
            <name>Apache 2.0</name>
            <url>http://www.apache.org/licenses/LICENSE-2.0</url>
        </license>
    </licenses>

    <dependencies>
        <!--依赖-->
    </dependencies>

		 <!--固定地址 是issue给你的地址-->
    <distributionManagement>
        <repository>
            <id>ossrh</id>
            <url>https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/</url>
        </repository>
        <snapshotRepository>
            <id>ossrh</id>
            <url>https://s01.oss.sonatype.org/content/repositories/snapshots</url>
        </snapshotRepository>
    </distributionManagement>


    <scm>
       <!--提交issue的地址 或者项目发布路径-->
        <url>https://github.com/lunasaw/luna-common</url>
      <!--git链接-->
        <connection>scm:git:https://github.com/lunasaw/luna-common.git</connection>
        <developerConnection>scm:git:https://github.com/lunasaw/luna-common</developerConnection>
    </scm>
    <profiles>
        <profile>
            <id>ossrh</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <build>
                <plugins>
                    <!--这是自动发布的插件-->
                    <plugin>
                        <groupId>org.sonatype.plugins</groupId>
                        <artifactId>nexus-staging-maven-plugin</artifactId>
                        <version>1.6.3</version>
                        <extensions>true</extensions>
                        <configuration>
                            <serverId>ossrh</serverId>
                            <nexusUrl>https://s01.oss.sonatype.org/</nexusUrl>
                            <autoReleaseAfterClose>true</autoReleaseAfterClose>
                        </configuration>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-release-plugin</artifactId>
                        <version>2.3.2</version>
                        <configuration>
                            <autoVersionSubmodules>true</autoVersionSubmodules>
                            <useReleaseProfile>false</useReleaseProfile>
                            <releaseProfiles>release</releaseProfiles>
                            <goals>deploy</goals>
                        </configuration>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-compiler-plugin</artifactId>
                        <version>3.0</version>
                        <configuration>
                            <source>1.8</source>
                            <target>1.8</target>
                        </configuration>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-gpg-plugin</artifactId>
                        <version>1.5</version>
                        <executions>
                            <execution>
                                <id>sign-artifacts</id>
                                <phase>verify</phase>
                                <goals>
                                    <goal>sign</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-source-plugin</artifactId>
                        <version>3.0.1</version>
                        <executions>
                            <execution>
                                <id>attach-sources</id>
                                <goals>
                                    <goal>jar-no-fork</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-javadoc-plugin</artifactId>
                        <version>2.9.1</version>
                        <configuration>
                            <javadocExecutable>${java.home}/../bin/javadoc</javadocExecutable>
                            <additionalparam>-Xdoclint:none</additionalparam>
                        </configuration>
                        <executions>
                            <execution>
                                <phase>package</phase>
                                <goals>
                                    <goal>jar</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
</project>
```

### 6. 发布jar包

执行**mvn clean deploy**处理，即可将jar包发布到Sonatype OSSRH仓库。

```bash
mvn clean deploy -projects fundebug-java,fundebug-spring 
```

我们的项目[fundebug-java-notifier](https://github.com/Fundebug/fundebug-java-notifier)含有多个模块，仅需部署fundebug-java与fundebug-spring，因此使用**-projects**选项来指定。

第一次执行**mvn clean deploy**命令时，需要输入GPG密钥的密码。

**mvn clean deploy**命令执行成功的输出是这样的(部分日志)：

```bash
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO]
[INFO] fundebug-java 0.2.0 ................................ SUCCESS [ 22.183 s]
[INFO] fundebug-spring 0.2.0 .............................. SUCCESS [ 16.383 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 38.728 s
[INFO] Finished at: 2019-01-12T20:10:16+08:00
[INFO] ------------------------------------------------------------------------
```

### 7. close并release

**mvn clean deploy**命令执行成功，使用JIRA账号登陆：https://s01.oss.sonatype.org/content/repositories/，就可以看到你所发布的jar包了：

![img](https://i.loli.net/2021/05/11/NbR75JLqEAhwkxD.png)

选中对于的repository之后，点击箭头所指的**close**，close时会检查发布的构件是否符合[要求](https://central.sonatype.org/pages/requirements.html)。若符合要求，则close成功，成功之后点击箭头所指的**release**，即可正式将jar包发布到Sonatype OSSRH仓库。

**release**成功大概2个小时之后，该构件就会同步到[Maven中央仓库](https://mvnrepository.com/)：

![img](https://image.fundebug.com/2019-01-12-fundebug-maven.png)

### 参考

- [Guide to uploading artifacts to the Central Repository](https://maven.apache.org/repository/guide-central-repository-upload.html)
- [OSSRH Guide](https://central.sonatype.org/pages/ossrh-guide.html)
- [Maven入门教程](https://blog.fundebug.com/2019/01/07/maven-tutorial/)
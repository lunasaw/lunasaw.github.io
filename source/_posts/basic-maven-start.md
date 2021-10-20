---
title: maven config start
date: 2020-07-27
banner_img: /img/basic/maven-binner.jpg
index_img: /img/basic/maven-index.jpg
tags: 
 - maven
categories:
 - basic-component
 - maven
---

# Maven配置学习

## 节点

pom.xml文件的节点大致可以分为以下几个部分：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
            http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
 
    <!-- 基本配置 -->
    <groupId>...</groupId>
    <artifactId>...</artifactId>
    <version>...</version>
    <packaging>...</packaging>
 
    <!-- 依赖配置 -->
    <dependencies>...</dependencies>
    <parent>...</parent>
    <dependencyManagement>...</dependencyManagement>
    <modules>...</modules>
    <properties>...</properties>
 
    <!-- 构建配置 -->
    <build>...</build>
    <reporting>...</reporting>
 
    <!-- 项目信息 -->
    <name>...</name>
    <description>...</description>
    <url>...</url>
    <inceptionYear>...</inceptionYear>
    <licenses>...</licenses>
    <organization>...</organization>
    <developers>...</developers>
    <contributors>...</contributors>
 
    <!-- 环境设置 -->
    <issueManagement>...</issueManagement>
    <ciManagement>...</ciManagement>
    <mailingLists>...</mailingLists>
    <scm>...</scm>
    <prerequisites>...</prerequisites>
    <repositories>...</repositories>
    <pluginRepositories>...</pluginRepositories>
    <distributionManagement>...</distributionManagement>
    <profiles>...</profiles>
</project>
```

pom.xml文件中，最外层的标签是`project`标签，该标签定义了如下属性：

> 1. `xmlns`属性：表明了该标签下元素的默认命名空间是：[http://maven.apache.org/POM/4.0.0](https://link.jianshu.com/?t=http%3A%2F%2Fmaven.apache.org%2FPOM%2F4.0.0)，xml文件中，定义命名空间的格式为：xmlns:namespace-prefix="namespaceURI"，而定义默认命名空间的格式为：xmlns="namespaceURI"；
> 2. `xmlns:xsi`属性：定义了一个namespace-prefix，其代表的命名空间URI为：[http://www.w3.org/2001/XMLSchema-instance](https://link.jianshu.com/?t=http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema-instance)，定义该命名空间主要是为了更方便的使用其所代表的命名空间中的schemaLocation属性；其中，使用xsi作为namespace-prefix，并不是硬性规定，只是一种通用的选择，当然也可以改成别的命名；
> 3. `xsi:schemaLocation`属性：该属性的使用格式为：`xsi:schemaLocation="namespaceURI1 schemaURI1 namespaceURI2 schemaURI2 ..."`，这里是说使用schemaURI1所对应的schema文件，校验命名空间namespaceURI1下的元素是否符合XML语法规范，后面的则是以此类推。

------

再来看一下基础信息的几个节点：

| 节点         | 对应的解释                                                   |
| ------------ | ------------------------------------------------------------ |
| modelVersion | pom模型版本，根据官方文档，Maven2和3只能为4.0.0              |
| groupId      | 项目组id，表明该项目所属的组织或公司，命名规则通常为组织或公司域名反转，然后再加项目名称 |
| artifactId   | 项目的id，有时候和项目名保持一致，有时候是`项目名 + 模块名`，该id是唯一的，一个goupId下面可能会有多个artifactId，就是通过artifactId区分。比如：`consumer-banking`。 |
| version      | 当前项目的版本号，一般是：`大版本.小版本.增量版本-限定版本号`，`SHAPSHOT`意为快照，说明该项目还处于开发中 |
| packaging    | 项目的打包方式，常用可选值：`pom, jar, ejb, maven-plugin, war, ear, rar, par`等，默认方式为jar |

### 项目信息

下面再来看下项目的一些信息：

| 节点          | 对应的解释                                                   |
| ------------- | ------------------------------------------------------------ |
| name          | 声明了一个对于用户更加友好的项目名称，非必须项，一般用于Maven生成的文档 |
| description   | 项目的详细描述信息，能使用HTML格式描述时不建议使用纯文本来描述， 一般用于Maven生成的文档 |
| url           | 项目主页的url，一般用于Maven生成的文档                       |
| inceptionYear | 项目开始的年份，一般是4位数字，涉及到介绍情况时用作提供版权信息 |
| licenses      | 当前项目所有的许可文件，每一个许可文件用一个许可元素来描述，然后描述额外的元素。 通常只列出适用于这个项目的许可文件，无需列出依赖项目的 许可文件列表。如果列出多个license，那么用户可以选择其中所需的，而不是接受所有的许可文件。 |
| organization  | 组织相关信息                                                 |
| developers    | 项目开发人员列表                                             |
| contributors  | 项目其他贡献者列表，同developers                             |

### licenses列表：

```xml
<license>    
    <!--license用于法律上的名称-->    
    <name>...</name>     
    <!--官方的license正文页面的URL-->    
    <url>....</url>
    <!--项目分发的主要方式：repo，可以从Maven库下载 manual， 用户必须手动下载和安装依赖-->    
    <distribution>repo</distribution>     
    <!--关于license的补充信息-->    
    <comments>....</comments>     
</license>
```

### organization属性：

```xml
<organization>
    <!-- 组织的主页 -->
    <url>...</url>
    <!-- 组织名称 -->
    <name>...</name>
</organization>
```

### developers属性：

```xml
<developers>  
    <!--某个开发者信息-->
    <developer>  
        <!--开发者的唯一标识符-->
        <id>....</id>  
        <!--开发者的全名-->
        <name>...</name>  
        <!--开发者的email-->
        <email>...</email>  
        <!--开发者的主页-->
        <url>...<url/>
        <!--开发者在项目中的角色-->
        <roles>  
            <role>Java Dev</role>  
            <role>Web UI</role>  
        </roles> 
        <!--开发者所属组织--> 
        <organization>sun</organization>  
        <!--开发者所属组织的URL-->
        <organizationUrl>...</organizationUrl>  
        <!--开发者属性，如即时消息如何处理等-->
        <properties>
        <!-- 和主标签中的properties一样，可以随意定义子标签 -->
        </properties> 
        <!--开发者所在时区， -11到12范围内的整数。--> 
        <timezone>-5</timezone>  
    </developer>  
</developers>
```

### contributors属性：

```xml
<contributors>
    <contributor>
        ...
    </contributor>
</contributors>
```

## 依赖配置

1. `dependencies`：项目相关依赖，如果父项目中的依赖，会被子项目引用，所以一般在父项目中定义子项目中共有的依赖。并且如果有需要，子项目可以修改所依赖包的版本：

```xml
<dependencies>
    <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
    </dependency>
</dependencies>
```

1. `parent`：用于确定父项目的坐标位置。

```xml
<parent>
    <groupId>com.learnPro</groupId>
    <artifactId>SIP-parent</artifactId>
    <relativePath></relativePath>
    <version>0.0.1-SNAPSHOT</version>
</parent>
```

> - groupId： 父项目的组Id标识符
> - artifactId：父项目的唯一标识符
> - relativePath：Maven首先在当前项目中找父项目的pom，然后在文件系统的这个位置（relativePath）查找，然后在本地仓库查找，再在远程仓库找。
> - version：父项目的版本

1. `dependencyManagement`：用于帮助管理children的依赖。例如如果parent使用`dependencyManagement`定义了一个`junit:junit4.0`，那么它的children就可以只引用 groupId和artifactId，而version就可以直接使用父模块的，如果有需要，我们也可以设置version，这样的好处就是可以集中管理依赖的详情，并且也更灵活。
     不过`dependencyManagement`里只是声明依赖，并不实现引入，因此子项目需要显示的声明需要用的依赖。配置同`dependencies`类似；

```xml
<!-- 子项目先通过parent继承之后，就可以引入使用了 -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
</dependency>
```

### dependencies几个重要的属性介绍：

> 1. type属性，默认为jar，指定pom，可以定义整个pom文件；
> 2. scope属性，当前包的依赖范围，用于限制依赖项的传递性，通过控制依赖的范围，可以指定该依赖在什么阶段有效。该属性共有5个值，默认是`compile`，也就是依赖关系在包含编译，测试，运行的所有类路径中都是可用的。而比如`provided`，则是依赖关系在编译和测试阶段的类路径中是可用的。如需查看全部，可参考：[Introduction to the Dependency Mechanism -Dependency Scope](https://link.jianshu.com/?t=http%3A%2F%2Fmaven.apache.org%2Fguides%2Fintroduction%2Fintroduction-to-dependency-mechanism.html)
> 3. optional属性，因为依赖是具有传递性的，例如 Project A 依赖于 Project B，B 依赖于 C，那么 B 对 C 的依赖关系也会传递给 A，如果我们不需要这种传递性依赖，就可以通过设置该属性的值。默认为false，即子项目默认都继承，true的话子项目必须显示的引入，与dependencyManagement里定义的依赖类似；
> 4. exclusions和exclusion属性，用于移除项目依赖相关，如果项目X需要A，而A包含B依赖，那么X可以声明不要B依赖，只要在exclusions中声明exclusion，将B从依赖树中删除即可。这一般用于解决jar包冲突的问题；

```xml
<dependencies>
    <dependency>
        <groupId>com.alibaba.china.shared</groupId>
        <artifactId>alibaba.apollo.webx</artifactId>
        <version>2.5.0</version>
        <exclusions>
          <exclusion>
            <artifactId>org.slf4j.slf4j-api</artifactId>
            <groupId>com.alibaba.external</groupId>
          </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

1. `modules`：Maven项目的一大特点就是以多模块著称，该标签就是用于指定当前项目所包含的模块，对该模块进行的Maven操作，会让所有子模块也进行相应的操作；

```xml
<modules>
   <module>com-a</>
   <module>com-b</>
   <module>com-c</>
<modules/>
```

1. `properties`：该标签用于定义pom文件中的常量，这样，在pom文件的任何地方，都可以通过`${java.version}`来引用该值；

```xml
<properties>
    <!-- 定义常量 -->
    <rocketmq.version>3.2.6</rocketmq.version>
</properties>
<!-- 使用 -->
<dependency>
    <groupId>com.alibaba.rocketmq</groupId>
    <artifactId>rocketmq-client</artifactId>
    <version>${rocketmq.version}</version>
</dependency>
```

## 构建配置

> 所谓的build，也就是项目构建。在构建配置的过程中，一般包含两个部分，一个是<build>，另一个是<reporting>；而在Maven的pom.xml中build又分为两种，一种被称为Project Build，也就是<project>下的直接子元素，另一种<build>被称为Profile Build，即是<profile>的直接子元素。Profile Build包含了基本的build元素，而Project Build还包含两个特殊的元素，即各种<...Directory>和<extensions>。

```xml
<build>
    <!--该元素设置了项目源码目录，当构建项目的时候，构建系统会编译目录里的源码。该路径是相对于pom.xml的相对路径。-->
    <sourceDirectory/>
    <!--该元素设置了项目脚本源码目录，该目录和源码目录不同：绝大多数情况下，该目录下的内容 会被拷贝到输出目录(因为脚本是被解释的，而不是被编译的)。-->
    <scriptSourceDirectory/>
    <!--该元素设置了项目单元测试使用的源码目录，当测试项目的时候，构建系统会编译目录里的源码。该路径是相对于pom.xml的相对路径。-->
    <testSourceDirectory/>
    <!--被编译过的应用程序class文件存放的目录。-->
    <outputDirectory/>
    <!--被编译过的测试class文件存放的目录。-->
    <testOutputDirectory/>
    <!--使用来自该项目的一系列构建扩展-->
    <extensions>
        <!--描述使用到的构建扩展。-->
        <extension>
            <!--构建扩展的groupId-->
            <groupId/>
            <!--构建扩展的artifactId-->
            <artifactId/>
            <!--构建扩展的版本-->
            <version/>
        </extension>
    </extensions>
    <!--当项目没有规定目标（Maven2 叫做阶段）时的默认值-->
    <defaultGoal/>
    <!--这个元素描述了项目相关的所有资源路径列表，例如和项目相关的属性文件，这些资源被包含在最终的打包文件里。-->
    <resources>
        <!--这个元素描述了项目相关或测试相关的所有资源路径-->
        <resource>
            <!-- 描述了资源的目标路径。该路径相对target/classes目录（例如${project.build.outputDirectory}）。举个例 子，如果你想资源在特定的包里(org.apache.maven.messages)，你就必须该元素设置为org/apache/maven /messages。然而，如果你只是想把资源放到源码目录结构里，就不需要该配置。-->
            <targetPath/>
            <!--是否使用参数值代替参数名。参数值取自properties元素或者文件里配置的属性，文件在filters元素里列出。-->
            <filtering/>
            <!--描述存放资源的目录，该路径相对POM路径-->
            <directory/>
            <!--包含的模式列表，例如**/*.xml.-->
            <includes/>
            <!--排除的模式列表，例如**/*.xml-->
            <excludes/>
        </resource>
    </resources>
    <!--这个元素描述了单元测试相关的所有资源路径，例如和单元测试相关的属性文件。-->
    <testResources>
        <!--这个元素描述了测试相关的所有资源路径，参见build/resources/resource元素的说明-->
        <testResource>
            <targetPath/>
            <filtering/>
            <directory/>
            <includes/>
            <excludes/>
        </testResource>
    </testResources>
    <!--构建产生的所有文件存放的目录-->
    <directory/>
    <!--产生的构件的文件名，默认值是${artifactId}-${version}。-->
    <finalName/>
    <!--当filtering开关打开时，使用到的过滤器属性文件列表-->
    <filters/>
    <!--子项目可以引用的默认插件信息。该插件配置项直到被引用时才会被解析或绑定到生命周期。给定插件的任何本地配置都会覆盖这里的配置-->
    <pluginManagement>
        <!--使用的插件列表 。-->
        <plugins>
            <!--plugin元素包含描述插件所需要的信息。-->
            <plugin>
                <!--插件在仓库里的group ID-->
                <groupId/>
                <!--插件在仓库里的artifact ID-->
                <artifactId/>
                <!--被使用的插件的版本（或版本范围）-->
                <version/>
                <!--是否从该插件下载Maven扩展（例如打包和类型处理器），由于性能原因，只有在真需要下载时，该元素才被设置成enabled。-->
                <extensions/>
                <!--在构建生命周期中执行一组目标的配置。每个目标可能有不同的配置。-->
                <executions>
                    <!--execution元素包含了插件执行需要的信息-->
                    <execution>
                        <!--执行目标的标识符，用于标识构建过程中的目标，或者匹配继承过程中需要合并的执行目标-->
                        <id/>
                        <!--绑定了目标的构建生命周期阶段，如果省略，目标会被绑定到源数据里配置的默认阶段-->
                        <phase/>
                        <!--配置的执行目标-->
                        <goals/>
                        <!--配置是否被传播到子POM-->
                        <inherited/>
                        <!--作为DOM对象的配置-->
                        <configuration/>
                    </execution>
                </executions>
                <!--项目引入插件所需要的额外依赖-->
                <dependencies>
                    <!--参见dependencies/dependency元素-->
                    <dependency>
                        ......
                    </dependency>
                </dependencies>
                <!--任何配置是否被传播到子项目-->
                <inherited/>
                <!--作为DOM对象的配置-->
                <configuration/>
            </plugin>
        </plugins>
    </pluginManagement>
    <!--使用的插件列表-->
    <plugins>
        <!--参见build/pluginManagement/plugins/plugin元素-->
        <plugin>
            <groupId/>
            <artifactId/>
            <version/>
            <extensions/>
            <executions>
                <execution>
                    <id/>
                    <phase/>
                    <goals/>
                    <inherited/>
                    <configuration/>
                </execution>
            </executions>
            <dependencies>
                <!--参见dependencies/dependency元素-->
                <dependency>
                    ......
                </dependency>
            </dependencies>
            <goals/>
            <inherited/>
            <configuration/>
        </plugin>
    </plugins>
</build>
```

reporting，该元素描述使用报表插件产生报表的规范。当用户执行“mvn site”，这些报表就会运行。 在页面导航栏能看到所有报表的链接。

```xml
<reporting>
    <!--true，则网站不包括默认的报表。这包括“项目信息”菜单中的报表。-->
    <excludeDefaults/>
    <!--所有产生的报表存放到哪里。默认值是${project.build.directory}/site。-->
    <outputDirectory/>
    <!--使用的报表插件和他们的配置。-->
    <plugins>
        <!--plugin元素包含描述报表插件需要的信息-->
        <plugin>
            <!--报表插件在仓库里的group ID-->
            <groupId/>
            <!--报表插件在仓库里的artifact ID-->
            <artifactId/>
            <!--被使用的报表插件的版本（或版本范围）-->
            <version/>
            <!--任何配置是否被传播到子项目-->
            <inherited/>
            <!--报表插件的配置-->
            <configuration/>
            <!--一组报表的多重规范，每个规范可能有不同的配置。一个规范（报表集）对应一个执行目标 。
              例如，有1，2，3，4，5，6，7，8，9个报表。1，2，5构成A报表集，对应一个执行目标。
              2，5，8构成B报表集，对应另一个执行目标-->
            <reportSets>
                <!--表示报表的一个集合，以及产生该集合的配置-->
                <reportSet>
                    <!--报表集合的唯一标识符，POM继承时用到-->
                    <id/>
                    <!--产生报表集合时，被使用的报表的配置-->
                    <configuration/>
                    <!--配置是否被继承到子POMs-->
                    <inherited/>
                    <!--这个集合里使用到哪些报表-->
                    <reports/>
                </reportSet>
            </reportSets>
        </plugin>
    </plugins>
</reporting>

```

## 环境配置

```xml
issueManagement：用于缺陷系统的跟踪与管理，比如Bugzilla, Jira等管理工具：
<issueManagement> 
    <system>Bugzilla</system> 
    <url>http://127.0.0.1/bugzilla/</url> 
</issueManagement> 
```

1. `ciManagement`：项目的持续集成信息：

```xml
<ciManagement>
    <system>continuum</system>
    <url>http://127.0.0.1:8080/continuum</url>
    <notifiers>
        <notifier>
            <type>mail</type>
            <sendOnError>true</sendOnError>
            <sendOnFailure>true</sendOnFailure>
            <sendOnSuccess>false</sendOnSuccess>
            <sendOnWarning>false</sendOnWarning>
            <address>continuum@127.0.0.1</address>
            <configuration></configuration>
        </notifier>
    </notifiers>
</ciManagement>
```

> 1. system：持续集成系统的名字
> 2. url：持续集成系统的URL
> 3. notifiers：构建完成时，需要通知的开发者/用户的配置项。包括被通知者信息和通知条件（错误，失败，成功，警告）

- type：通知方式
- sendOnError：错误时是否通知
- sendOnFailure：失败时是否通知
- sendOnSuccess：成功时是否通知
- sendOnWarning：警告时是否通知
- address：通知发送到的地址
- configuration：扩展项

1. `mailingLists`：项目相关邮件列表：

>    1. subscribe, unsubscribe: 订阅邮件（取消订阅）的地址或链接，如果是邮件地址，创建文档时，链接会被自动创建；
>    2. archive：浏览邮件信息的URL；
>     3. post：接收邮件的地址；

1. `SCM`：(Source Control Management)，该标签允许你配置你的代码库，供Maven Web站点和其他插件使用。

```xml
<scm>
    <connection>scm:svn:http://127.0.0.1/svn/my-project</connection>
    <developerConnection>scm:svn:https://127.0.0.1/svn/my-project</developerConnection>
    <tag>HEAD</tag>
    <url>http://127.0.0.1/websvn/my-project</url>
</scm>
```

- connection, developerConnection：这两个表示我们如何连接到maven的版本库。connection只提供读，developerConnection将提供写的请求；写法如：`scm:[provider]:[provider_specific]`，如果连接到CVS仓库，可以配置如下：
  `scm:cvs:pserver:127.0.0.1:/cvs/root:my-project`
- tag：项目标签，默认HEAD
- url：共有仓库路径

1. `prerequisites`：项目构建的前提。

> 1. repositories，pluginRepositories，依赖和扩展的远程仓库列表；pom里面的仓库与setting.xml里的仓库功能是一样的。主要的区别在于，pom里的仓库是个性化的。比如一家大公司里的setting文件是公用 的，所有项目都用一个setting文件，但各个子项目却会引用不同的第三方库，所以就需要在pom里设置自己需要的仓库地址。而pluginRepositories，与Repositories具有类似的结构，只是Repositories是dependencies的home，而这个是plugins 的home。`

```xml
<repositories>
    <repository>
      <releases>
        <enabled>false</enabled>
        <updatePolicy>always</updatePolicy>
        <checksumPolicy>warn</checksumPolicy>
      </releases>
      <snapshots>
        <enabled>true</enabled>
        <updatePolicy>never</updatePolicy>
        <checksumPolicy>fail</checksumPolicy>
      </snapshots>
      <id>codehausSnapshots</id>
      <name>Codehaus Snapshots</name>
      <url>http://snapshots.maven.codehaus.org/maven2</url>
      <layout>default</layout>
    </repository>
  </repositories>
  <pluginRepositories>
    ...
  </pluginRepositories>
```

- releases，snapshots：这是各种构件的策略，release或者snapshot，这两个集合，POM就可以根据独立仓库任意类型的依赖改变策略。如：一个人可能只激活下载snapshot用来开发。
- enable：true或者false，决定仓库是否对于各自的类型激活(release 或者 snapshot)。
- updatePolicy: 这个元素决定更新频率。maven将比较本地pom的时间戳（存储在仓库的maven数据文件中）和远程的。有以下选择: always, daily (默认), interval:X (x是代表分钟的整型) ， never。
- checksumPolicy：当Maven向仓库部署文件的时候，它也部署了相应的校验和文件。可选的为：ignore，fail，warn，或者不正确的校验和。
- layout：在上面描述仓库的时候，提到他们有统一的布局。Maven 2有它仓库默认布局。然而，Maven 1.x有不同布局。使用这个元素来表明它是default还是legacy。

1. `profiles`：POM 4.0的一个新特性是一个项目能够根据所构建的环境改变设置。通过profiles，我们可以指定构建的环境是开发，测试，UAT，生产等环境；
2. 我们可以通过activation标签来激活相应的profile，也就是指定要构建的默认的环境，并且可以配置各种需要的环境，并且激活profile的方式也有多种，具体参数可查看官网文档。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      https://maven.apache.org/xsd/maven-4.0.0.xsd">
  ...
  <profiles>
    <profile>
      <id>test</id>
      <activation>
        <activeByDefault>false</activeByDefault>
        <jdk>1.5</jdk>
        <os>
          <name>Windows XP</name>
          <family>Windows</family>
          <arch>x86</arch>
          <version>5.1.2600</version>
        </os>
        <property>
          <name>sparrow-type</name>
          <value>African</value>
        </property>
        <file>
          <exists>${basedir}/file2.properties</exists>
          <missing>${basedir}/file1.properties</missing>
        </file>
      </activation>
      ...
    </profile>
  </profiles>
</project>

```


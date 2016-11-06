title: Maven 学习笔记
date: 2015/11/16 23:39:00
updated: 2015/11/19 01:01:00
tag: 
	- Maven
	- 笔记
---

## Maven 简介

Maven 的作用：

- 构建工具
- 依赖管理
- 项目信息

还可以用来进行极限编程（XP）和敏捷开发、瀑布开发。

## Maven 的安装和配置

### 安装

- 下载Maven 3
- 解压安装
- 配置环境变量
	- `%M2_HOME%`
	- `path = %M2_HOME%/bin`

在Mac下，可以直接使用BrewHome的`brew install`来安装。

<!--more-->

#### 安装目录分析

- bin
- boot
- conf
- lib

bin：包含了`mvn`运行脚本（还有`mvnDebug`）。

boot：不必关心。

conf：非常重要的配置文件`settings.xml`。最好把该文件复制到`~/.m2/settings.xml`下来配置，这样利于管理、恢复和升级。

lib: Maven运行时得所有Java类库，可以说此目录就是真正的Maven。

### IDE

//TODO 

通过`mvn -v`来查看Maven、Java、OS的版本信息等。

## Maven 使用入门

### POM

- XML 头`<?xml version = "1.0" encoding = "UTF-8">`
- 根节点`<project>`
	- `<modelVersion>`只能选择4.0.0
	- `<groupId>`项目的地址，例如`com.googlecode.myapp`
	- `<artifactId>`项目的名称，例如`hello-world`
	- `<version>`版本号，默认起始为`1.0-SNAPSHOT`
	- `<name>`项目描述

### 代码的目录结构

- src
	- main
		- java
		- resources
	- test
		- java
- target
	- classes
	- test-classes
	- .jar
- pom.xml

要求`src`目录必须按照如上的结构生成，其中源代码全部放在`src/main/java`目录下，测试代码全部放在`src/test/java`目录下，项目的根目录下必须有`pom.xml`文件。

`target`目录是由Maven命令脚本生成的，所有的输出文件都会在该目录下。

### 最常用的四个Maven命令

- `mvn clean compile`
- `mvn clean test`
- `mvn clean package`
- `mvn clean install`

这其中，`install`>`package`>`test`>`compile`会形成包含关系，执行左边的命令之前会先执行右边的命令。

命令都在项目的根部目录下执行。

- `compile`会编译`src/main/java`下的所有源代码，输出`.class`文件到`target/classes`目录下。
- `test`会编译`src/test/java`下的所有代码，输出`.class`文件到`target/test-classes`目录下。
- `package`会把编译结果打包为`artifactId-version.jar`输出到`target`目录下。
- `install`会把打包结果安装到本地仓库，供其他Maven项目使用。

#### 关于测试

如果在`pom.xml`中添加了Junit依赖：

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.10</version>
        <scope>test</scope>
        <!--junit中声明的scope只对test目录下的文件有效-->
    </dependency>
</dependencies>
```

则可以在测试代码中利用junit来进行代码测试。

使用`@test`来标记测试的方法。

#### 关于打包

默认的`package`命令并不包含`.jar`中的`mainClass`属性，需要通过`maven-shade-plugin`插件来指定主类。

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>1.2.1</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <transformers>
                            <transformer implementation = "org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>com.chenkun.mvnstudy.helloworld.HelloWorld</mainClass>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### 项目骨架

可以使用Archetype来生成如之前所述的代码目录结构的项目骨架。

```mvn
mvn archetype:generate
```

### IDE

//TODO

## Maven 坐标

Maven的任何一个构建都需要一个坐标来定位，它们是通过一些元素来定义的：`groupId`、`artifactId`、`version`、`packaging`、`classifier`。

- `<groupId>`: 定义当前Maven项目隶属的实际项目，其表示方式与Java包名的表示方式类似，通常与反向域名一一对应，例如`org.sonatype.nexus`。
- `<artifactId>`: 定义实际项目中的一个Maven项目（模块），推荐做法是将实际项目作为前缀，例如`nexus-indexer`，这样便于快速检索和区分。
- `<version>`: 使用规范的版本定义，包括对`SNAPSHOT`的使用。
- `<packaging>`: 定义Maven项目的打包方式，默认为`.jar`。
- `<classifier>`: 帮助定义构建输出的一些附属构件，例如`nexus-indexer-2.0.0-javadoc.jar`等，此属性不能直接定义，需由插件生成。

> 项目构件的文件名是与坐标相对应的，一般的规则是`artifactID-version[-classifier].packaging`。

## 依赖

### 依赖的配置

```xml
<project>
	...
	<dependencies>
		<dependency>
			<groupId>...</groupId>
			<artifactId>...</artifactId>
			<version>...</version>
			<type>...</type>
			<scope>...</scope>
			<optional>...</optional>
			<exclusions>
				<exclusion>...</exclusion>
				...
			</exclusions>
		</dependency>
		...
	</dependencies>
	...
</project>
```

这其中：

- `<groupId>`、`<artifactId>`和`<version>`是依赖的基本坐标，**唯一且重要**。
- `<type>`对应的是坐标定义的`<packaging>`，默认的值是`.jar`。
- `<scope>`依赖的生效范围。
- `<optional>`标记以来是否可选。
- `<exclusions>`用来排除传递性依赖。

### 依赖的范围

- `compile`
- `test`
- `provided`
- `runtime`
- `system`

依赖范围与生效的classpath关系如下：

|依赖范围（Scope）|对于编译classpath有效|对于测试classpath有效|对于运行时classpath有效|例子|
|:--:|:--:|:--:|:--:|:--:|
|compile|Y|Y|Y|spring-core|
|test|-|Y|-|JUnit|
|provided|Y|Y|-|servlet-api|
|runtime|-|Y|Y|JDBC驱动实现|
|system|Y|Y|-|本地的，Maven之外|

### 依赖的传递性

A依赖于B，B依赖于C，称**A对B是第一直接依赖，B对C是第二直接依赖，A对C是传递性依赖**。

### 依赖调解

如果存在这样的依赖关系：

- A->B->C->X(1.0)
- A->D->X(2.0)

X是A的传递性依赖，关于到底选择哪一个依赖有如下两条调解规则：

1. 路径最近者优先
2. 第一声明者优先

### 可选依赖

如果存在如下依赖：

A->B->X or Y （optional）

那么**A将不会依赖X，也不会依赖Y**。由于可选依赖一定程度违反了**单一职责性原则**，所以并不推荐使用。

### 排除依赖

如果存在以下依赖：

A->B->C(version?)

A可以通过排除依赖C来接触和C的传递依赖关系，然后重新进行直接依赖定义C(version)。

### 归类依赖

多个相关依赖如果使用相同的版本号，可以在`<dependencies>`之外定于如下的`<property>`来统一定义：

```xml
<properties>
	<name.version>x.y.z</name.version>
</properties>
```

后面的依赖在调用版本号时便可直接通过`${name.version}`来调用`x.y.z`这个版本了，这样非常利于管理和升级依赖包。

### 优化依赖

```bash
mvn dependency:list
mvn dependency:tree
```

命令可以用来查看当前项目中的**依赖列表**和**依赖树**，同时表明依赖范围等

```bash
mvn dependency:analyze
```

则可以分析依赖中出现的问题：

- Used undeclared dependencies
- Unused declared dependencies

其中`Used undecalred dependencies`需要注意仔细分析。

## 仓库

### 仓库的分类

- Maven 仓库
	- 本地仓库
	- 远程仓库
		- 中央仓库
		- 私服
		- 其他公共仓库

### 本地仓库

本地仓库在`~/.m2/repository`目录下，任何一个构件，**只有当被下载安装到了本地仓库中才会被其他的Maven项目所使用**。所以，如果要使用本地的构件作为依赖，必须先使用`mvn clean install`来安装到本地仓库。

如果因为某些原因想要改变本地仓库的存储目录，修改`~/.m2/settings.xml`即可（此文件需从Maven的`config`目录下拷贝过来）。

```xml
<settings>
	<localRepository>new path</localRepository>
</settings>
```

### 远程仓库

当本地仓库没有我们所需要的构件的时候，Maven就会到远程仓库去下载。

> 这好比藏书。例如，我想要读『红楼梦』，会先检查自己的书房是否已经收藏了这本书，如果发现没有这本书，于是就跑去书店买一本回来，放到书房里。可能有一天我又想读一本英文版的『程序员修炼之道』，而书房里只有中文版，于是又去书店找，可发现书店没有，好在还有网上书店，于是从Amazon买了一本，几天后我收到了这本书，又放到了自己的书房。

> 本地仓库和远程仓库之间的关系就好比是自己的书房和网上书店的关系、自己想看的书先从书房找、没有的话从网上淘一本放到自己的书房看、自己的书房最需要一个就够了、而网上书店可以有很多、同样本地仓库只有一个、而远程仓库就可以有很多。我们可以在`settings.xml`中指定远程仓库。

#### 中央仓库

中央仓库是Maven默认的远程仓库，它的地址是`http://repo.maven.org/maven2`。中央仓库在超级POM中是这样描述的：

```xml
<repository>
	<id>central</id>
	<name>Central Repository</name>
	<url>https://repo.maven.apache.org/maven2</url>
	<layout>default</layout>
	<snapshots>
		<enabled>false</enabled>
	</snapshots>
</repository>
```

#### 私服

私服是一种特殊的远程仓库，它一般架设在内网中，一方面它可以将别的远程仓库的构件缓存下来，一方面可以提供构件给内部Maven用户下载，同时还支持本地用户上传自己的构件供网内的用户使用，大大提高了访问速度和减轻了中央仓库的压力。常见的架构私服的软件是**Nexus**。

### 远程仓库的配置

在很多情况下，默认的中央仓库可能无法满足项目的需求（例如有些依赖`JDBC`因为版权许可原因并不能放在中央仓库里），项目可能存在别的公共远程仓库里，那我们就可以在自己的项目中配置仓库。下面以添加一个`JBoss`仓库为例：

```xml
<project>
	...
	<repository>
		<id>jboss</id>
		<name>JBoss Repository</name>
		<url>https://repository.jboss.com/maven2</url>
		<layout>default</layout>
		<releases>
			<enabled>true</enabled>
		</releases>
		<snapshots>
			<enabled>false</enabled>
		</snapshots>
	</repository>
	...
</project>
```

> 注意：Maven的中央仓库的`<id>`是`central`，如果其他仓库的声明使用了这个id，就会覆盖中央仓库。

如果允许`<snapshot>`快照版本的构件被索引下载，那么有可能使得项目的某些功能变得不太稳定，这个属性需要看需求来开启。

### 远程仓库的认证

大部分的仓库访问都不需要认证，但是出于安全考虑，有些仓库也是需要认证保护，特别内部使用的仓库。访问这样的仓库，需要**提前配置认证信息**。认证信息的配置必须在`setting.xml`中，这样是更加安全的，下例是给一个`id`为`my-repo`的仓库配置的认证信息：

```xml
<settings>
	...
	<servers>
		<server>
			<id>my-repo</id>
			<username>repo-user</username>
			<password>repo-pwd</password>
		</server>
		...
	</servers>
	...
</settings>
```

### 快照版本

SNAPSHOT 版本的构件在上传至仓库时，会被打上`x.y.z-yyyymmdd.hhmmss-times`的时间戳例如`2.1-20151119.145320-13`，代表该构件是2.1版本，2015年11月19日14时53分20秒第13次提交更新。

这样别人使用快照版的构件是都将得到最新的版本。

> 快照版本只应在组织内部的项目或模块间依赖使用，因为这时，组织对这些快照版本的依赖具有完全的理解及控制权。项目不应该依赖于任何组织外部的快照版本依赖，由于快照版本的不稳定性，这样的依赖会造成潜在的危险。也就是说，**即使构建项目今天是成功的，由于外部的快照版本依赖实际对应的构件随时可能变化，项目的构建就有可能由于这些外部的不可控因素而失败**。

### 镜像

如果仓库X中能提供所有Y仓库能提供的构件，那么就称X是Y的一个镜像。举个例子，`http://maven.net.cn/content/groups/public`是中央仓库`http://repo.maven.org/maven2`在中国的一个镜像，国内的访问速度普遍会快上很多。因此，可以配置`setting.xml`文件使用该镜像来替代中央仓库：

```xml
<settings>
	...
	<mirror>
     	<id>maven.oschina.net</id>
     	<name>one of the central mirrors in China</name>
     	<url>http://maven.oschina.net/content/groups/public/</url>
     	<mirrorOf>central</mirrorOf>
    </mirror>
	...
</settings>
```

关于镜像的常见用法是配合私服，也就是使得私服是所有远程仓库的镜像，只需`<mirrorOf>`的值为`*`即可。**这样对于任何远程仓库的请求都会映射到私服对应的`<url>`上去**。

为了满足一些复杂的需求，Maven还支持更高级的镜像配置：

- `*`：匹配所有的远程仓库
- `external：*`：匹配所有远程仓库，使用`localhost`的除外
- `repo1, repo2`：使用逗号隔开多个远程仓库
- `*, !repo1`：匹配除`repo1`以外所有的远程仓库

### 仓库搜索服务

- http://repository.sonatype.org/
- http://www.jarvana.com/jarvana/
- http://www.mvnbrower.com/
- http://mvnrepository.com/
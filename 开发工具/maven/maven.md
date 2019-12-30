## 创建 Maven 项目
` mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false `
## Maven 目录结构
```properties
my-app
|-- pom.xml
`-- src
    |-- main
    |   `-- java
    |       `-- com
    |           `-- mycompany
    |               `-- app
    |                   `-- App.java
    `-- test
        `-- java
            `-- com
                `-- mycompany
                    `-- app
                        `-- AppTest.java
```
## POM 文件
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>
 
  <name>Maven Quick Start Archetype</name>
  <url>http://maven.apache.org</url>
 
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.8.2</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```
## 编译项目
### 打包命令
```sh
mvn package
```
### 输出
```sh
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2 seconds
[INFO] Finished at: Thu Jul 07 21:34:52 CEST 2011
[INFO] Final Memory: 3M/6M
[INFO] ------------------------------------------------------------------------
```
### 运行
```sh
java -cp target/my-app-1.0-SNAPSHOT.jar com.mycompany.app.App
```

# Tools
### 生命周期
- validate: validate the project is correct and all necessary information is available
- compile: compile the source code of the project
- test: test the compiled source code using a suitable unit testing framework. These tests should not require the code be packaged or deployed
- package: take the compiled code and package it in its distributable format, such as a JAR.
- integration-test: process and deploy the package if necessary into an environment where integration tests can be run
- verify: run any checks to verify the package is valid and meets quality criteria
- install: install the package into the local repository, for use as a dependency in other projects locally
- deploy: done in an integration or release environment, copies the final package to the remote repository for sharing with other developers and projects.
> 除了上面的默认列表之外，还有两个其他的Maven生命周期

- clean: cleans up artifacts created by prior builds
- site: generates site documentation for this project

> maven 的生命周期会按顺序执行

`mvn clean dependency:copy-dependencies package`

### 生成站点
`mvn site`
> 生成的文档会存在于 **target/site** 文件夹下

# 内部插件
### 1. 编译
`mvn compile`

> 输出

```sh
[INFO] ----------------------------------------------------------------------------
[INFO] Building Maven Quick Start Archetype
[INFO]    task-segment: [compile]
[INFO] ----------------------------------------------------------------------------
[INFO] artifact org.apache.maven.plugins:maven-resources-plugin: \
  checking for updates from central
...
[INFO] artifact org.apache.maven.plugins:maven-compiler-plugin: \
  checking for updates from central
...
[INFO] [resources:resources]
...
[INFO] [compiler:compile]
Compiling 1 source file to <dir>/my-app/target/classes
[INFO] ----------------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] ----------------------------------------------------------------------------
[INFO] Total time: 3 minutes 54 seconds
[INFO] Finished at: Fri Sep 23 15:48:34 GMT-05:00 2005
[INFO] Final Memory: 2M/6M
[INFO] ----------------------------------------------------------------------------
```

> 第一次运行该命令时，maven 会需要下载依赖的插件。
> 编译后的文件存放在 **${basedir}/target/classes** 目录下

### 1. 运行测试

`mvn test`

> 执行上面的命令会得到一下的输出

```sh
[INFO] ----------------------------------------------------------------------------
[INFO] Building Maven Quick Start Archetype
[INFO]    task-segment: [test]
[INFO] ----------------------------------------------------------------------------
[INFO] artifact org.apache.maven.plugins:maven-surefire-plugin: \
  checking for updates from central
...
[INFO] [resources:resources]
[INFO] [compiler:compile]
[INFO] Nothing to compile - all classes are up to date
[INFO] [resources:testResources]
[INFO] [compiler:testCompile]
Compiling 1 source file to C:\Test\Maven2\test\my-app\target\test-classes
...
[INFO] [surefire:test]
[INFO] Setting reports dir: C:\Test\Maven2\test\my-app\target/surefire-reports
 
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
[surefire] Running com.mycompany.app.AppTest
[surefire] Tests run: 1, Failures: 0, Errors: 0, Time elapsed: 0 sec
 
Results :
[surefire] Tests run: 1, Failures: 0, Errors: 0
 
[INFO] ----------------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] ----------------------------------------------------------------------------
[INFO] Total time: 15 seconds
[INFO] Finished at: Thu Oct 06 08:12:17 MDT 2005
[INFO] Final Memory: 2M/8M
[INFO] ----------------------------------------------------------------------------
```
> 如果你想只编译测验代码而不去运行，你可以使用下面的命令
`mvn test-compile`

### 3. 创建 jar 并安装到本地仓库
制作一个 jar 包非常简单，只需要执行下面的命令

`mvn package`

> 如果你查找过 POM 文件，就会发现 POM 文件中为 **packaging** 元素被设置为 **jar** , 这就是为什么上面的命令会产生一个 jar 包的原因。
> 你可以在 **${basedir}/target** 目录下找到生成的 JAR 文件。 

如果你想把刚刚生成的文件安装到本地仓库（**${user.home}/.m2/repository** 是默认的仓库地址）

`mvn install`

执行上面的命令会输出以下信息

```sh
[INFO] ----------------------------------------------------------------------------
[INFO] Building Maven Quick Start Archetype
[INFO]    task-segment: [install]
[INFO] ----------------------------------------------------------------------------
[INFO] [resources:resources]
[INFO] [compiler:compile]
Compiling 1 source file to <dir>/my-app/target/classes
[INFO] [resources:testResources]
[INFO] [compiler:testCompile]
Compiling 1 source file to <dir>/my-app/target/test-classes
[INFO] [surefire:test]
[INFO] Setting reports dir: <dir>/my-app/target/surefire-reports
 
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
[surefire] Running com.mycompany.app.AppTest
[surefire] Tests run: 1, Failures: 0, Errors: 0, Time elapsed: 0.001 sec
 
Results :
[surefire] Tests run: 1, Failures: 0, Errors: 0
 
[INFO] [jar:jar]
[INFO] Building jar: <dir>/my-app/target/my-app-1.0-SNAPSHOT.jar
[INFO] [install:install]
[INFO] Installing <dir>/my-app/target/my-app-1.0-SNAPSHOT.jar to \
   <local-repository>/com/mycompany/app/my-app/1.0-SNAPSHOT/my-app-1.0-SNAPSHOT.jar
[INFO] ----------------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] ----------------------------------------------------------------------------
[INFO] Total time: 5 seconds
[INFO] Finished at: Tue Oct 04 13:20:32 GMT-05:00 2005
[INFO] Final Memory: 3M/8M
[INFO] ----------------------------------------------------------------------------
```

> 请注意，surefire插件（执行测试）使用特定的命名约定查找包含在文件中的测试，默认包含的测试规则有：

- **/*Test.java
- **/Test*.java
- **/*TestCase.java

> 默认排除的有：

- **/Abstract*Test.java
- **/Abstract*TestCase.java

## SNAPSHOT 是什么版本?

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  ...
  <groupId>...</groupId>
  <artifactId>my-app</artifactId>
  ...
  <version>1.0-SNAPSHOT</version>
  <name>Maven Quick Start Archetype</name>
  ...
```

> **SNAPSHOT** 值是指沿开发分支的“最新”代码，并不保证代码稳定或不变，相反，“发布”版本中的代码（没有后缀SNAPSHOT的任何版本值）不变。

> 换句话说，SNAPSHOT版本是最终“发布”版本之前的“开发”版本。 SNAPSHOT比它的发布“更老”

> 在发布过程中，x.y-SNAPSHOT的一个版本更改为x.y. 发布过程也将开发版本增加到x.(y + 1)-SNAPSHOT。例如，版本1.0-SNAPSHOT作为版本1.0发布，而新的开发版本是版本1.1-SNAPSHOT。

## 怎么使用插件
无论何时您想为Maven项目定制构建，都可以通过添加或重新配置插件来完成。

在这个例子中，我们将配置Java编译器以允许JDK 5.0源代码。这与将此添加到您的POM一样简单：

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.3</version>
      <configuration>
        <source>1.5</source>
        <target>1.5</target>
      </configuration>
    </plugin>
  </plugins>
</build>
```

> **configuration** 元素会应用在 compile 插件编译的每一个元素上面。

## 怎么过滤资源文件
有时候资源文件需要包含一个只能在构建时提供的值。为了在Maven中实现这一点，请使用语法$ {<property name\>}将引用包含值的属性引用到资源文件中。

该属性可以是您的pom.xml中定义的值之一，在用户的settings.xml中定义的值，在外部属性文件中定义的属性或系统属性。

要在复制时使用Maven过滤器资源，只需将pom.xml中的资源目录的过滤设置为true即可：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>
 
  <name>Maven Quick Start Archetype</name>
  <url>http://maven.apache.org</url>
 
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
 
  <build>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
  </build>
</project>
```

您会注意到我们必须添加之前没有的 **build**，**resources** 和 **resource** 元素。另外，我们必须明确说明这些资源位于 **src/main/resources** 目录中。所有这些信息之前都是作为默认值提供的，但由于过滤的默认值为false，我们必须将其添加到我们的pom.xml中，以覆盖该默认值并将筛选设置为true。

为了引用在您的pom.xml中定义的属性，属性名称使用定义值的XML元素的名称，允许“pom”作为项目（root）元素的别名。所以 **${project.name}** 指的是项目的名称，**${project.version}** 指的是项目的版本，**${project.build.finalName}** 指的是构建项目时创建的文件的最终名称被打包等等。注意，POM的一些元素具有默认值，所以不需要在你的pom.xml中明确地定义这些值在这里可用。类似地，用户的settings.xml中的值可以使用以“settings”开头的属性名称来引用（例如，**${settings.localRepository}** 引用用户本地存储库的路径）。

为了继续我们的例子，让我们添加一些属性到 **application.properties** 文件中（我们把它放在 **src/main/resources** 目录中），这些文件的值将在资源被过滤时提供：

```properties
# application.properties
application.name=${project.name}
application.version=${project.version}
```

有了这个，您可以执行以下命令（process-resources是资源被复制和过滤的构建生命周期阶段）：

```sh
mvn process-resources
```

target/classes下的application.properties文件（并最终进入jar的文件）如下所示：

```properties
# application.properties
application.name=Maven Quick Start Archetype
application.version=1.0-SNAPSHOT
```

要引用在外部文件中定义的属性，您只需在pom.xml中添加对此外部文件的引用。首先，我们创建我们的外部属性文件，并将其称为 src/main/filters/filter.properties：

```properties
# filter.properties
my.filter.value=hello!
```

接下来，我们将在pom.xml中添加对这个新文件的引用：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>
 
  <name>Maven Quick Start Archetype</name>
  <url>http://maven.apache.org</url>
 
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
 
  <build>
    <filters>
      <filter>src/main/filters/filter.properties</filter>
    </filters>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
  </build>
</project>
```

然后，如果我们在application.properties文件中添加对此属性的引用：

```properties
# application.properties
application.name=${project.name}
application.version=${project.version}
message=${my.filter.value}
```

mvn process-resources命令的下一次执行会将我们的新属性值放入application.properties中。作为在外部文件中定义my.filter.value属性的替代方法，您也可以在pom.xml的properties部分中定义它，并获得相同的效果（注意我不需要引用src/main/filters/filter.properties）：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>
 
  <name>Maven Quick Start Archetype</name>
  <url>http://maven.apache.org</url>
 
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
 
  <build>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
  </build>
 
  <properties>
    <my.filter.value>hello</my.filter.value>
  </properties>
</project>
```

过滤资源还可以从系统属性获取值; 可以是内置于Java中的系统属性（如java.version或user.home），也可以是使用标准Java -D参数在命令行上定义的属性。为了继续这个例子，让我们把我们的application.properties文件改为如下所示：

```properties
# application.properties
java.version=${java.version}
command.line.prop=${command.line.prop}
```

现在，当您执行以下命令（请注意命令行中command.line.prop属性的定义）时，application.properties文件将包含系统属性中的值。

```sh
mvn process-resources "-Dcommand.line.prop=hello again"
```

## 依赖

```xml
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
```

对于每个外部依赖关系，您至少需要定义4种东西：groupId，artifactId，版本和范围。groupId，artifactId和version与构建该依赖项目的项目的pom.xml中给出的相同。scope元素指示项目如何使用该依赖关系，并且可以是诸如 **compile**，**test** 和 **runtime** 的值。

maven 会根据 groupId、artifactId 和 version 去本地仓库(${user.home/.m2/repository})中寻找, 如果在本地仓库中找不到时， 则会从 maven 中央仓库下载对应的依赖到本地。

## 部署到远程仓库
要将jar部署到外部存储库，必须在pom.xml中配置存储库URL，并在settings.xml中配置连接到存储库的认证信息。

以下是使用scp和用户名/密码认证的示例：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>
 
  <name>Maven Quick Start Archetype</name>
  <url>http://maven.apache.org</url>
 
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.codehaus.plexus</groupId>
      <artifactId>plexus-utils</artifactId>
      <version>1.0.4</version>
    </dependency>
  </dependencies>
 
  <build>
    <filters>
      <filter>src/main/filters/filters.properties</filter>
    </filters>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
  </build>
  <!--
   |
   |
   |
   -->
  <distributionManagement>
    <repository>
      <id>mycompany-repository</id>
      <name>MyCompany Repository</name>
      <url>scp://repository.mycompany.com/repository/maven2</url>
    </repository>
  </distributionManagement>
</project>
```

___

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      http://maven.apache.org/xsd/settings-1.0.0.xsd">
  ...
  <servers>
    <server>
      <id>mycompany-repository</id>
      <username>jvanzyl</username>
      <!-- Default value is ~/.ssh/id_dsa -->
      <privateKey>/path/to/identity</privateKey> (default is ~/.ssh/id_dsa)
      <passphrase>my_key_passphrase</passphrase>
    </server>
  </servers>
  ...
</settings>
```

> 请注意，如果您正在连接到sshd_confing中参数“PasswordAuthentication”设置为“no”的openssh ssh服务器，则您必须每次输入密码以进行用户名/密码身份验证（尽管您可以使用另一个ssh登录客户端通过输入用户名和密码）。在这种情况下，您可能想切换到公钥认证。

> 如果在settings.xml中使用密码，应该小心。

## 创建文档
为了让您能够从Maven的文档系统入手，您可以使用以下命令使用原型机制为现有项目生成一个站点：

```sh
mvn archetype:generate \
  -DarchetypeGroupId=org.apache.maven.archetypes \
  -DarchetypeArtifactId=maven-archetype-site \
  -DgroupId=com.mycompany.app \
  -DartifactId=my-app-site
```

## 构建其他类型的项目
生命周期适用于任何项目类型。我们可以创建一个简单的Web应用程序：

```sh
mvn archetype:generate \
    -DarchetypeGroupId=org.apache.maven.archetypes \
    -DarchetypeArtifactId=maven-archetype-webapp \
    -DgroupId=com.mycompany.app \
    -DartifactId=my-webapp
```

请注意，这些都必须在一行上。这将创建一个名为my-webapp的目录，其中包含以下项目描述符：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-webapp</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>
 
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
 
  <build>
    <finalName>my-webapp</finalName>
  </build>
</project>
```

请注意<packaging>元素 - 这会告诉Maven构建为WAR。转到webapp项目的目录并尝试：

```sh
mvn package
```

你会看到target / my-webapp.war被构建，并且执行了所有常规步骤。

## 一次构建多个项目
Maven内置了处理多个模块的概念。在本节中，我们将展示如何构建上面的WAR，并在一个步骤中包含以前的JAR。
首先，我们需要在其他两个目录上面添加一个父pom.xml文件，所以它应该看起来像这样：

```sh
+- pom.xml
+- my-app
| +- pom.xml
| +- src
|   +- main
|     +- java
+- my-webapp
| +- pom.xml
| +- src
|   +- main
|     +- webapp
```


您要创建的POM文件应包含以下内容：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.mycompany.app</groupId>
  <artifactId>app</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>pom</packaging>
 
  <modules>
    <module>my-app</module>
    <module>my-webapp</module>
  </modules>
</project>
```

我们需要从Web应用程序依赖JAR，所以将其添加到my-webapp/pom.xml中：

```xml
  ...
  <dependencies>
    <dependency>
      <groupId>com.mycompany.app</groupId>
      <artifactId>my-app</artifactId>
      <version>1.0-SNAPSHOT</version>
    </dependency>
    ...
  </dependencies>
```

最后，将下面的<parent>元素添加到子目录中的其他pom.xml文件中：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <parent>
    <groupId>com.mycompany.app</groupId>
    <artifactId>app</artifactId>
    <version>1.0-SNAPSHOT</version>
  </parent>
  ...
```

现在，尝试一下...从顶层目录运行：

```sh
mvn verify
```

现在已经在my-webapp/target/my-webapp.war中创建了WAR ，并且包含了JAR：

```sh
$ jar tvf my-webapp/target/my-webapp-1.0-SNAPSHOT.war
   0 Fri Jun 24 10:59:56 EST 2005 META-INF/
 222 Fri Jun 24 10:59:54 EST 2005 META-INF/MANIFEST.MF
   0 Fri Jun 24 10:59:56 EST 2005 META-INF/maven/
   0 Fri Jun 24 10:59:56 EST 2005 META-INF/maven/com.mycompany.app/
   0 Fri Jun 24 10:59:56 EST 2005 META-INF/maven/com.mycompany.app/my-webapp/
3239 Fri Jun 24 10:59:56 EST 2005 META-INF/maven/com.mycompany.app/my-webapp/pom.xml
   0 Fri Jun 24 10:59:56 EST 2005 WEB-INF/
 215 Fri Jun 24 10:59:56 EST 2005 WEB-INF/web.xml
 123 Fri Jun 24 10:59:56 EST 2005 META-INF/maven/com.mycompany.app/my-webapp/pom.properties
  52 Fri Jun 24 10:59:56 EST 2005 index.jsp
   0 Fri Jun 24 10:59:56 EST 2005 WEB-INF/lib/
2713 Fri Jun 24 10:59:56 EST 2005 WEB-INF/lib/my-app-1.0-SNAPSHOT.jar
```

这个怎么用？首先，创建的父POM（称为app）包含pom和定义的模块列表。这告诉Maven运行所有对项目集合的操作，而不仅仅是当前操作（重写此行为，可以使用--non-recursive命令行选项）。

接下来，我们告诉WAR它需要my-app JAR。这样做有一些好处：它可以在类路径中使用WAR中的任何代码（在本例中为none），它确保JAR始终在WAR之前构建，并且指示WAR插件将JAR包含在它的库目录。

你可能已经注意到junit-4.11.jar是一个依赖项，但并没有在WAR中结束。其原因是<scope> test </ scope>元素 - 它仅用于测试，因此不包含在Web应用程序中，因为编译时间依赖性my-app是。

最后一步是包含父母定义。这与Maven 1.0中可能熟悉的扩展元素不同：这可以确保即使通过在存储库中查找项目与父项分开分发项目，也可以始终定位POM。

## Super POM

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <name>Maven Default Project</name>
 
  <repositories>
    <repository>
      <id>central</id>
      <name>Maven Repository Switchboard</name>
      <layout>default</layout>
      <url>http://repo1.maven.org/maven2</url>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>
 
  <pluginRepositories>
    <pluginRepository>
      <id>central</id>
      <name>Maven Plugin Repository</name>
      <url>http://repo1.maven.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <releases>
        <updatePolicy>never</updatePolicy>
      </releases>
    </pluginRepository>
  </pluginRepositories>
 
  <build>
    <directory>${project.basedir}/target</directory>
    <outputDirectory>${project.build.directory}/classes</outputDirectory>
    <finalName>${project.artifactId}-${project.version}</finalName>
    <testOutputDirectory>${project.build.directory}/test-classes</testOutputDirectory>
    <sourceDirectory>${project.basedir}/src/main/java</sourceDirectory>
    <!-- TODO: MNG-3731 maven-plugin-tools-api < 2.4.4 expect this to be relative... -->
    <scriptSourceDirectory>src/main/scripts</scriptSourceDirectory>
    <testSourceDirectory>${project.basedir}/src/test/java</testSourceDirectory>
    <resources>
      <resource>
        <directory>${project.basedir}/src/main/resources</directory>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>${project.basedir}/src/test/resources</directory>
      </testResource>
    </testResources>
   <pluginManagement>
       <plugins>
         <plugin>
           <artifactId>maven-antrun-plugin</artifactId>
           <version>1.3</version>
         </plugin>       
         <plugin>
           <artifactId>maven-assembly-plugin</artifactId>
           <version>2.2-beta-2</version>
         </plugin>         
         <plugin>
           <artifactId>maven-clean-plugin</artifactId>
           <version>2.2</version>
         </plugin>
         <plugin>
           <artifactId>maven-compiler-plugin</artifactId>
           <version>2.0.2</version>
         </plugin>
         <plugin>
           <artifactId>maven-dependency-plugin</artifactId>
           <version>2.0</version>
         </plugin>
         <plugin>
           <artifactId>maven-deploy-plugin</artifactId>
           <version>2.4</version>
         </plugin>
         <plugin>
           <artifactId>maven-ear-plugin</artifactId>
           <version>2.3.1</version>
         </plugin>
         <plugin>
           <artifactId>maven-ejb-plugin</artifactId>
           <version>2.1</version>
         </plugin>
         <plugin>
           <artifactId>maven-install-plugin</artifactId>
           <version>2.2</version>
         </plugin>
         <plugin>
           <artifactId>maven-jar-plugin</artifactId>
           <version>2.2</version>
         </plugin>
         <plugin>
           <artifactId>maven-javadoc-plugin</artifactId>
           <version>2.5</version>
         </plugin>
         <plugin>
           <artifactId>maven-plugin-plugin</artifactId>
           <version>2.4.3</version>
         </plugin>
         <plugin>
           <artifactId>maven-rar-plugin</artifactId>
           <version>2.2</version>
         </plugin>        
         <plugin>                
           <artifactId>maven-release-plugin</artifactId>
           <version>2.0-beta-8</version>
         </plugin>
         <plugin>                
           <artifactId>maven-resources-plugin</artifactId>
           <version>2.3</version>
         </plugin>
         <plugin>
           <artifactId>maven-site-plugin</artifactId>
           <version>2.0-beta-7</version>
         </plugin>
         <plugin>
           <artifactId>maven-source-plugin</artifactId>
           <version>2.0.4</version>
         </plugin>         
         <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.4.3</version>
         </plugin>
         <plugin>
           <artifactId>maven-war-plugin</artifactId>
           <version>2.1-alpha-2</version>
         </plugin>
       </plugins>
     </pluginManagement>
  </build>
 
  <reporting>
    <outputDirectory>${project.build.directory}/site</outputDirectory>
  </reporting>
  <profiles>
    <profile>
      <id>release-profile</id>
 
      <activation>
        <property>
          <name>performRelease</name>
          <value>true</value>
        </property>
      </activation>
 
      <build>
        <plugins>
          <plugin>
            <inherited>true</inherited>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-sources</id>
                <goals>
                  <goal>jar</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-javadoc-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-javadocs</id>
                <goals>
                  <goal>jar</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-deploy-plugin</artifactId>
            <configuration>
              <updateReleaseInfo>true</updateReleaseInfo>
            </configuration>
          </plugin>
        </plugins>
      </build>
    </profile>
  </profiles>
 
</project>
```

## Minimal POM

POM 的最低要求如下:

- 项目根
- modelVersion - 应该设置为4.0.0
- groupId - 项目组的标识。
- artifactId - 工件的ID（项目）
- 版本 - 指定组下的工件版本

```xml
<project>
	  <modelVersion>4.0.0</modelVersion>
	  <groupId>com.mycompany.app</groupId>
	  <artifactId>my-app</artifactId>
	  <version>1</version>
</project>
```

> POM要求配置其groupId，artifactId和版本。这三个值构成项目完全限定的工件名称。这是<groupId>：<artifactId>：<version>的形式。至于上面的例子，其完全限定的工件名称是“com.mycompany.app:my-app:1”。

> 另外，如第一部分所述，如果未指定配置详细信息，Maven将使用其默认值。其中一个默认值是包装类型。每个Maven项目都有一个包装类型。如果没有在POM中指定，那么将使用默认值“jar”。

> 此外，正如你所看到的，在最小的POM中，存储库没有被指定。如果使用最小的POM构建项目，它将继承Super POM中的存储库配置。因此，当Maven在最小的POM中看到依赖关系时，它会知道这些依赖关系将从Super POM中指定的http://repo.maven.apache.org/maven2下载。


## 项目继承

POM 中合并的元素如下:

- 依赖
- 开发者和贡献者
- 插件列表（包括报告）
- 插件执行与匹配的ID
- 插件配置
- 资源

#####案例一：子项目在父项目节点下

```sh
.
 |-- my-module
 |   `-- pom.xml
 `-- pom.xml
```

子项目中添加 <parent\> 节点

```xml
<project>
  <parent>
    <groupId>com.mycompany.app</groupId>
    <artifactId>my-app</artifactId>
    <version>1</version>
  </parent>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-module</artifactId>
  <version>1</version>
</project>
```

因为府项目内帮我们管理了 groupId 及 version，所有上面的代码可简写为：

```xml
<project>
  <parent>
    <groupId>com.mycompany.app</groupId>
    <artifactId>my-app</artifactId>
    <version>1</version>
  </parent>
  <modelVersion>4.0.0</modelVersion>
  <artifactId>my-module</artifactId>
</project>
```

#####案例二：子项目不在父项目节点下

	.
	 |-- my-module
	 |   `-- pom.xml
	 `-- parent
	     `-- pom.xml

为了解决这个目录结构（或任何其他目录结构），我们必须将<relativePath>元素添加到我们的父节。

```xml
<project>
  <parent>
    <groupId>com.mycompany.app</groupId>
    <artifactId>my-app</artifactId>
    <version>1</version>
    <relativePath>../parent/pom.xml</relativePath>
  </parent>
  <modelVersion>4.0.0</modelVersion>
  <artifactId>my-module</artifactId>
</project>
```

## 项目聚合

项目聚合类似于项目继承。但不是从模块中指定父POM，而是从父POM指定模块。通过这样做，父项目现在知道它的模块，并且如果对父项目调用Maven命令，那么Maven命令也会执行到父项模块。要执行项目聚合，您必须执行以下操作：

- 将父级POM包装更改为“pom”值。
- 在父POM中指定其模块的目录（子POM）

#### 案例一：子项目在父项目节点下

	.
	 |-- my-module
	 |   `-- pom.xml
	 `-- pom.xml

如果我们要将我的模块聚合到我的应用程序中，我们只需要修改我的应用程序。

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1</version>
  <packaging>pom</packaging>
 
  <modules>
    <module>my-module</module>
  </modules>
</project>
```

### 案例二：子项目不在父节点下

	.
	 |-- my-module
	 |   `-- pom.xml
	 `-- parent
	     `-- pom.xml

解决方案是指定模块的路径

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1</version>
  <packaging>pom</packaging>
 
  <modules>
    <module>../my-module</module>
  </modules>
</project>
```


## 项目继承与项目聚合

如果您有多个Maven项目，并且它们都具有相似的配置，则可以通过抽出相似的配置并制作父项目来重构项目。因此，您所要做的就是让您的Maven项目继承该父项目，然后将这些配置应用于所有这些项目。

如果您有一组构建或一起处理的项目，您可以创建一个父项目并让该父项目将这些项目声明为其模块。通过这样做，您只需构建父级，其余的将随之而来。

但是，当然，您可以同时拥有项目继承和项目聚合。意思是说，你可以让你的模块指定一个父项目，同时让这个父项目指定这些Maven项目作为它的模块。你只需要应用所有三条规则：

- 在每个子POM中指定他们的父POM是谁。
- 将父级POM <packaging\>更改为“pom”值。
- 在父POM中指定其模块的目录（子POM）

项目结构：

	.
	 |-- my-module
	 |   `-- pom.xml
	 `-- parent
	     `-- pom.xml

父项目：

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1</version>
  <packaging>pom</packaging>
 
  <modules>
    <module>../my-module</module>
  </modules>
</project>
```

子项目：

```xml
<project>
  <parent>
    <groupId>com.mycompany.app</groupId>
    <artifactId>my-app</artifactId>
    <version>1</version>
    <relativePath>../parent/pom.xml</relativePath>
  </parent>
  <modelVersion>4.0.0</modelVersion>
  <artifactId>my-module</artifactId>
</project>
```

## 项目插值和变量

Maven鼓励的做法之一是不要重复自己。但是，在某些情况下，您需要在多个不同位置使用相同的值。为了帮助确保值仅指定一次，Maven允许您在POM中使用自己的和预定义的变量。

例如，要访问project.version变量，可以这样引用它：

 	<version>${project.version}</version>

> 需要注意的一个因素是，如上所述，继承后会处理这些变量。这意味着如果一个父项目使用一个变量，那么它在孩子中的定义，而不是父项，将是最终使用的定义。

##### 可用变量
###### 项目模型变量

作为单个值元素的模型的任何字段都可以作为变量引用。例如，$ {project.groupId}，$ {project.version}，$ {project.build.sourceDirectory}等等。请参阅POM参考以查看完整的属性列表。

这些变量都由前缀“project”引用。你也可以参照pom。作为前缀，或者完全省略前缀 - 这些表单现在已被弃用，不应使用。

###### 特殊变量

	project.basedir			当前项目所在的目录。
	project.baseUri			当前项目所在的目录，表示为URI。由于Maven 2.1.0
	maven.build.timestamp	表示构建开始的时间戳。由于Maven 2.1.0-M1

构建时间戳的格式可以通过声明属性maven.build.timestamp.format来定制，如下例所示：

```xml
<project>
  ...
  <properties>
    <maven.build.timestamp.format>yyyy-MM-dd'T'HH:mm:ss'Z'</maven.build.timestamp.format>
  </properties>
  ...
</project>
```

> 格式模式必须符合SimpleDateFormat的API文档中给出的规则。如果该属性不存在，则格式默认为示例中已经给出的值。

###### 属性

您也可以将项目中定义的任何属性作为变量进行引用。考虑下面的例子：

```xml
<project>
  ...
  <properties>
    <mavenVersion>2.1</mavenVersion>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.apache.maven</groupId>
      <artifactId>maven-artifact</artifactId>
      <version>${mavenVersion}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.maven</groupId>
      <artifactId>maven-project</artifactId>
      <version>${mavenVersion}</version>
    </dependency>
  </dependencies>
  ...
</project>
```
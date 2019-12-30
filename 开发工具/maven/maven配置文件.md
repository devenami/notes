## .mvn 文件夹

`.mvn`文件夹位于当前项目的根文件夹下面。该文件夹内可以包含`extensions.xml`、`maven.config`和`jvm.config`。

### extensions.xml

`extension.xml`用于定义maven项目启动所需的额外库。

如果不适用这个文件的话，就需要将我们需要的jar包放在maven安装路径的`/lib/ext`目录下，或者使用`mvn -Dmaven.ext.class.path=extension.jar`的方式，总之这样很不方便。

文件格式如下：

```xml
<extensions xmlns="http://maven.apache.org/EXTENSIONS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/EXTENSIONS/1.0.0 http://maven.apache.org/xsd/core-extensions-1.0.0.xsd">
  <extension>
    <groupId/>
    <artifactId/>
    <version/>
  </extension>
</extensions>
```

### maven.config

`maven.config`文件可以存储在使用maven命令时，额外添加的选项，如`-T3 -U --fail-at-end`，我们使用以前的方式，就需要使用`mvn -T3 -U --fail-at-end clean package`。这样我们每次在执行maven命令时，可能就会忘记添加选项。`maven.config`就为我们解决了这种问题，通过将自定义选项配置到`maven.config`文件内，就不需要在每次执行命令时额外添加自定义选项，maven会自动帮我们加上。并且，这种方式，对于多模块化项目也是同样适用的。

```sh
-Drevision=2.0.0-SNAPSHOT -T3 -U --fail-at-end
```

### jvm.config

`jvm.config`文件用于定义maven启动时jvm的选项，这意味着您可以在每个项目基础上定义构建选项。例如下面的配置：

```
-Xmx2048m -Xms1024m -XX:MaxPermSize=512m -Djava.awt.headless=true
```

这样，我们就不需要再配置`MAVEN_OPTS`、`.mavenrc`文件了。



## settings.xml

`settings.xml`用于自定义maven的配置，每个项目的`pom.xml`也可以用于配置maven，但是，`pom.xml`是只针对于单个项目的，而`settings.xml`中配置的属性是针对于全局。

`settings.xml`文件存在于一下两个位置：

* maven的安装目录：${maven.home}/conf/settings.xml
* 用户目录：${user.home}/.m2/settings.xml

如果以上两个文件同时存在，maven会合并两个文件中配置的值，如果遇到冲突的值，则用户目录下的配置优先于maven安装目录下的配置。

### 简单属性

**\<localRepository>${user.home}/.m2/repository\</localRepository>**：指定本地jar包存储的位置。

**\<interactiveMode>true\</interactiveMode>**：是都可以和用户的输入进行交互。

**\<offline>false\</offline>**：是否启用离线默认。

### Plugin Groups

此元素包含一个plugingroup元素列表，每个元素都包含一个groupid。当使用插件且命令行中未提供groupid时，将搜索该列表。此列表自动包含org.apache.maven.plugins和org.codehaus.mojo。

```xml
  <pluginGroups>
    <pluginGroup>org.eclipse.jetty</pluginGroup>
  </pluginGroups>
```

我们在命令行中执行`mvn org.eclipse.jetty:jetty-maven-plugin:run`，如果给定上面的配置，则可以修改为：`mvn jetty:run`

### Servers

用于下载和部署的存储库由POM的存储库和分发管理元素定义。但是，某些设置（如用户名和密码）不应与pom.xml一起分发。此类型的信息应存在于settings.xml中的生成服务器上。

```xml
<servers>
    <server>
      <id>server001</id>
      <username>my_login</username>
      <password>my_password</password>
      <privateKey>${user.home}/.ssh/id_dsa</privateKey>
      <passphrase>some_passphrase</passphrase>
      <filePermissions>664</filePermissions>
      <directoryPermissions>775</directoryPermissions>
      <configuration></configuration>
    </server>
  </servers>
```

### Mirrors

用于配置maven私服，如果没有配置，默认使用国外的中心仓库。

```xml
  <mirrors>
    <mirror>
      <id>planetmirror.com</id>
      <name>PlanetMirror Australia</name>
      <url>http://downloads.planetmirror.com/pub/maven2</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
  </mirrors>
```

**id, name**：当前镜像唯一的id和友好的名称，ID用于区分镜像元素，并在连接到镜像时从<servers>部分选择相应的凭据。

**url**：此镜像的基URL。生成系统将使用此URL连接到存储库，而不是原始存储库URL。

**mirrirOf**：这是其镜像的存储库的ID。例如，要指向maven中央存储库的镜像（https://repo.maven.apache.org/maven2/），请将此元素设置为central。更高级的映射，如repo1、repo2或*，！也可以使用。这不能与镜像ID匹配。

### Proxies

用于配置代理服务器

```xml
  <proxies>
    <proxy>
      <id>myproxy</id>
      <active>true</active>
      <protocol>http</protocol>
      <host>proxy.somewhere.com</host>
      <port>8080</port>
      <username>proxyuser</username>
      <password>somepassword</password>
      <nonProxyHosts>*.google.com|ibiblio.org</nonProxyHosts>
    </proxy>
  </proxies>
```

**id**：当前代理唯一的id

**active**：是否启用该代理

**protocol**, **host**, **port**：protocol://host:port 

**nonProxyHosts**：这是不应代理的主机列表。列表的分隔符是代理服务器的预期类型；上面的示例是管道分隔-逗号分隔也是常见的。

### Profiles

settings.xml中的profile元素是pom.xml profile元素的截断版本。它由激活、存储库、插件存储库和属性元素组成。配置文件元素只包含这四个元素，因为它们与整个构建系统（settings.xml文件的角色）有关，而与单个项目对象模型设置无关。

如果一个配置文件在设置中处于活动状态，它的值将覆盖pom或profiles.xml文件中任何等同的id配置文件。

#### Activation

激活(activation)是配置文件(profiles)的关键。与POM的概要文件一样，概要文件的强大功能来自它仅在特定情况下修改某些值的能力；这些情况是通过激活元素指定的。

```xml
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
          <name>mavenVersion</name>
          <value>2.0.3</value>
        </property>
        <file>
          <exists>${basedir}/file2.properties</exists>
          <missing>${basedir}/file1.properties</missing>
        </file>
      </activation>
      ...
    </profile>
  </profiles>
```

activation在满足所有指定条件时启用，但并非同时需要所有条件。activation的条件有如下：

* **jdk**：激活在JDK元素中有内置的、以Java为中心的检查。如果在与给定前缀匹配的JDK版本号下运行测试，则此选项将激活。在上面的示例中，1.5.0_06将匹配。从Maven 2.1开始也支持范围。
* **os**：OS元素可以定义上面所示的一些操作系统特定的属性。
* **property**：如果maven检测到相应name=value对的属性（一个可以在pom中被$name取消引用的值），配置文件将激活。
* **file**：给定的文件名可以通过文件的存在来激活配置文件，或者如果文件丢失。

activation元素并不是只有满足上面的任一条件的方式才能激活，也可以在命令行中使用`-P`参数指定profile的id的形式启用。如 `-P test`， 多个之间使用逗号分隔。

#### Properties

Maven属性是值占位符，类似于Ant中的属性。它们的值可以通过符号$x在POM中的任何位置访问，其中x是属性。它们有五种不同的样式，都可以从settings.xml文件访问：

* **env.x**：在变量前加上“env.”将返回shell的环境变量。例如，${env.path}包含$path环境变量（Windows中为%path%）。
* **project.x**:pom中的点（.）标记路径将包含相应元素的值。例如：\<project>\<version>1.0\<version>\<project>可以通过${project.version}获得。
* settings.x:settings.xml中带点（.）符号的路径将包含相应元素的值。例如：\<settings>\<offline>false.\<offline>\<settings>可以通过${settings.offline}获得。
* Java系统属性：通过java.lang.System.getProperties() 访问的所有属性都可用作POM属性，例如${java.home}
* x：在\<properties/>元素或外部文件中设置，该值可以用作${somevar}。

```xml
  <profiles>
    <profile>
      ...
      <properties>
        <user.install>${user.home}/our-project</user.install>
      </properties>
      ...
    </profile>
  </profiles>
```

如果此配置文件处于活动状态，则可以从POM访问属性${user.install}

#### Repositories

存储库是Maven用来填充构建系统本地存储库的项目的远程集合。Maven就是从这个本地存储库中调用它的插件和依赖项的。不同的远程存储库可能包含不同的项目，在被激活的\<profile>下，可以搜索它们以查找匹配的版本或快照工件。主要用于将当前项目**部署到远程库**。

```xml
  <profiles>
    <profile>
      ...
      <repositories>
        <repository>
          <id>codehausSnapshots</id>
          <name>Codehaus Snapshots</name>
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
          <url>http://snapshots.maven.codehaus.org/maven2</url>
          <layout>default</layout>
        </repository>
      </repositories>
      <pluginRepositories>
        ...
      </pluginRepositories>
      ...
    </profile>
  </profiles>
```

* **releases**, **snapshots** : 这些是每种类型的工件、发布或快照的策略。有了这两个集合，POM就可以在单个存储库中独立地更改每种类型的策略。例如，可以决定只启用快照下载，可能是为了开发目的。
* **enabled** : 是否启用相应的类型。
* **updatePolicy** ： 此元素指定尝试更新的频率。Maven将本地POM的时间戳（存储在存储库的Maven元数据文件中）与远程POM进行比较。选项包括：始终(always)、每天(daily)（默认）、间隔：x (interval:X )（其中x是以分钟为单位的整数）或从不(never)。
* **checksumPolicy** : 当Maven将文件部署到存储库时，它还部署相应的校验和文件。您的选项是忽略(ignore)、失败(fail)或警告丢失(warn on missing)或不正确(incorrect)的校验和。
* **layout** : 在上面对存储库的描述中，提到它们都遵循一个公共布局。这基本上是正确的。Maven2有其存储库的默认布局；但是，Maven1.x有不同的布局。使用此元素指定它是默认的(default )还是传统的(legacy)。

#### Plugin Repositories

maven插件部署的仓库，内部元素类似于\<repositories>

### Active Profiles

```xml
<activeProfiles>
    <activeProfile>env-test</activeProfile>
  </activeProfiles>
```

这包含一组\<activeProfile>元素，每个元素都有一个profile id的值。定义为\<activeProfile>的任何profile id都将处于活动状态，而不管环境设置如何。如果没有找到匹配的配置文件，则不会发生任何情况。


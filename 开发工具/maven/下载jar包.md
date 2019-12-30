## 编写pom文件

我们通过`pom.xml`文件来声明需要下载的jar包。

`pom.xml`

```xml
<?xml version="1.0"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.feb13th</groupId>
    <artifactId>download-jar</artifactId>
    <version>1.0</version>
    <dependencies>

        <!-- 需要下载什么jar包 添加相应依赖 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.58</version>
        </dependency>

    </dependencies>
</project>
```

## 执行命令

在当前`pom`文件同级目录下打开命令行。

输入如下命令下载jar

```shell
mvn -f pom.xml dependency:copy-dependencies
```

脚本执行完成后会在`./target/dependency/`目录下展示所有依赖的jar包文件。

## 编写windows执行脚本

我们可以将该命令封装为一个脚本，通过使用脚本的方式来下载jar。

`run.bat`

```
call mvn -f pom.xml dependency:copy-dependencies
@pause
```

项目结构为：

```
maven:
	pom.xml
	run.bat
```


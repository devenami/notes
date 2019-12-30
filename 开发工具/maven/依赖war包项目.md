## 依赖war包项目

在项目的开发过程中，多个项目之间可以存在依赖关系，而项目的打包格式都是`war`包，如果使用`jar`打包格式，就无法部署在web容器中。

解决方案就是在项目打包时，同时打包一个`jar`包进行兼容。

### 在被依赖项目中进行配置

```xml
<plugin>
    <artifactId>maven-war-plugin</artifactId>
    <version>3.2.1</version>
    <configuration>
        <!-- 通过配置该下方的属性进行打包为jar -->
        <attachClasses>true</attachClasses>
        <failOnMissingWebXml>false</failOnMissingWebXml>
        <!-- 打包的jar的文件后缀 项目名-version-classifier.jar -->
        <classifier>classes</classifier>
    </configuration>
</plugin>
```

### 依赖打包的jar

```xml
<dependency>
    <groupId>com.**</groupId>
    <artifactId>myserver-api</artifactId>
    <version>${project.version}</version>
    <type>jar</type>
    <!-- 这里必须和被依赖项目的classifier一致 -->
    <classifier>classes</classifier>
</dependency>
```


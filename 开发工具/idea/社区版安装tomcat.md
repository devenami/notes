## idea社区版安装tomcat

idea社区版中默认不存在`tomcat`的插件，如果我们项目中需要使用`tomcat`作为`web服务器`的话，则需要我们手动进行安装插件。

在社区版中安装`tomcat`插件的方式有两种，一种是使用`smart tomcat`插件，另一种是使用`maven`的`tomcat8-maven-plugin`插件。

### smart tomcat 安装方式

#### 安装插件

1. 通过`settings`的`plugins`仓库搜索`smart tomcat`进行安装，安装完成后重启idea
2. 选择`run`按钮旁边的下拉框，点击`Edit Configurations`
3. 弹出的`run/debug configurations`配置框中点击左上角的`+`号按钮。选择`Smart Tomcat`

#### smart tomcat 配置文件

| 名称          | 解释                                        |
| ------------- | ------------------------------------------- |
| Name          | 当前启动服务运行时名称                      |
| Tomcat Server | 通过后面的`...`按钮选择本地tomcat的安装路径 |
| Deployment    | `webapp`在项目中的路径                      |
| Context Path  | 部署项目的上下文路径                        |
| Server Port   | 服务器端口                                  |
| AJP port      | 保持默认                                    |
| Tomcat Port   | 保持默认                                    |
| VM options    | JVM运行参数                                 |
| Env options   | tomcat 运行环境参数                         |

#### 启动

选择`run`按钮旁边下拉框中刚刚配置的`Name`,使用`run`或`debug`进行运行。

### maven配置运行tomcat

#### 配置`pom.xml`

```xml
<build> 
    <finalName>appName</finalName> 
    <plugins> 
      <plugin> 
        <groupId>org.apache.tomcat.maven</groupId> 
        <artifactId>tomcat7-maven-plugin</artifactId> 
        <version>2.1</version> 
        <configuration> 
          <port>9090</port> 
          <path>/</path> 
          <uriEncoding>UTF-8</uriEncoding> 
          <server>tomcat7</server> 
        </configuration> 
      </plugin> 
    </plugins> 
 </build> 
```

#### 配置运行方式

1. 选择`run`按钮旁边的下拉框，点击`Edit Configurations`
2. 弹出的`run/debug configurations`配置框中点击左上角的`+`号按钮。选择`Maven`

#### maven `Paramters` 配置

| 名称              | 解释        |
| ----------------- | ----------- |
| Working directory | 项目根目录  |
| Command line      | tomcat7:run |

#### 启动

选择`run`按钮旁边下拉框中刚刚配置的`Name`,使用`run`或`debug`进行运行。
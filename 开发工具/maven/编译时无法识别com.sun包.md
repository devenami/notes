### 问题

项目中使用了`com.sun`包下的类，该包是sun公司不对外开源的代码。使用`mvn compile` 命令进行编译时报错:

`javac uses a special symbol table that does not include all Sun-proprietary classes. When javac is compiling code it doesn't link against rt.jar by default. Instead it uses special symbol file lib/ct.sym with class stubs.`

相应的java源码文件中也会出现红线。

### 原因

正如报错内容，javac默认编译代码时，并没有直接使用`rt.jar`文件，而是使用了一个特殊的符号表，这个符号表并没有包含所有sun私有化的class。使用这个特殊连接文件`lib/ct.sym`替换了`rt.jar`。

### 解决方案

我们需要通知maven编译插件忽略这个特殊的连接文件。所以我们需要在编译插件执行之前，传入`-XDignore.symbol.file` 命令，手动通知编译器忽略默认的连接文件，使用`rt.jar`。

```xml
<plugins>
	<plugin>
		<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-compiler-plugin</artifactId>
            <version>${version}</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <!-- 添加启动命令，类似于 -->
                <!-- javac -XDignore.symbol.file -->
                <compilerArgs>
                    <arg>-XDignore.symbol.file</arg>
                </compilerArgs>
                <fork>true</fork>
            </configuration>
    </plugin>
</plugins>
```


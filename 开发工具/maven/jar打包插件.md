### 此插件打包为jar，并对MANIFEST.MF作修改：`mvn package`：

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-jar-plugin</artifactId>
	<version>2.4</version>
	<configuration>
		<archive>
			<manifest>
				<!-- 为依赖包添加路径, 这些路径会写在MANIFEST文件的Class-Path下 -->
				<addClasspath>true</addClasspath>
				<classpathPrefix>lib/</classpathPrefix>
				<!-- jar启动入口类 -->
				<mainClass>com.lwt.test.Test</mainClass>
			</manifest>
			<manifestEntries>
				<!-- 在MANIFEST.MF下添加项，其实所有项都可以在此添加而不用上面的manifest标签 -->
				<!-- conf/会被添加到Class-Path -->
				<Class-Path>conf/</Class-Path>
			</manifestEntries>
		</archive>
		<includes>
			<!-- 打jar包时，只打包class文件。若想打包配置文件，注释掉 -->
			<include>**/*.class</include>
		</includes>
	</configuration>
</plugin>
```

### 将依赖包解压并打包为一个单独jar包的插件：`mvn assembly:assembly`

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-assembly-plugin</artifactId>
	<version>2.2</version>
	<configuration>
		<archive>
			<manifest>
				<mainClass>com.lwt.test.Test</mainClass>
			</manifest>
		</archive>
		<descriptorRefs>
			<descriptorRef>
				jar-with-dependencies
			</descriptorRef>
		</descriptorRefs>
	</configuration>
</plugin>
```

### 打包为jar并生成依赖的目录，里面存放依赖jar：`mvn package`

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-jar-plugin</artifactId>
	<version>2.4</version>
	<configuration>
		<archive>
			<manifest>
				<addClasspath>true</addClasspath>
				<classpathPrefix>dependency/</classpathPrefix>
				<mainClass>com.lwt.test.Test</mainClass>
			</manifest>
		</archive>
	</configuration>
</plugin>

<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-dependency-plugin</artifactId>
	<version>2.10</version>
	<executions>
		<execution>
			<id>copy-dependencies</id>
			<phase>package</phase>
			<goals>
				<goal>copy-dependencies</goal>
			</goals>
		</execution>
	</executions>
</plugin>
```

### 此插件能生成可执行批处理文件：`mvn package appassembler:assemble`

```xml
<plugin>
	<groupId>org.codehaus.mojo</groupId>
	<artifactId>appassembler-maven-plugin</artifactId>
	<version>1.10</version>
	<configuration>
		<programs>
			<program>
				<mainClass>com.lwt.test.Test</mainClass>
				<id>app</id>
			</program>
		</programs>
	</configuration>
</plugin>
```

### 引用

[maven：打包jar的插件](https://blog.csdn.net/xuejianbest/article/details/84859145)


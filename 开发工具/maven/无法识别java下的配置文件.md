### 问题

我们在`src/main/java`下添加了自定义的`xml`文件，当使用idea或maven进行编译时，`classes`目录下并没有包含指定的`xml`配置文件。

### 问题原因

maven默认认为`src/main/java`为`*.java`存储的目录，maven会使用`maven-compiler-plugin`插件编译该目录，将该目录下所有的`.java`文件编译为`.class`文件后，`src/main/java`文件夹就没有存在的必要了。maven在打包是会默认将`src/main/java`下的文件忽略。

### 解决方案

通过在`<build>`标签下使用`<resources>`标签手动更改`maven`的文件包含策略。

#### 方案1、包含指定目录下的所有文件

```xml
<build>
	<resources>
        <!-- 指定包含 src/main/java 目录下的所有文件 -->
		<resource>
			<directory>src/main/java</directory>
			<includes>
				<include>**/*.*</include>
			</includes>
		</resource>
        <!-- 指定包含 src/main/resources 目录下的所有文件 -->
        <resource>
			<directory>src/main/resources</directory>
			<includes>
				<include>**/*.*</include>
			</includes>
		</resource>
	</resources>
</build>
```

#### 方案2、包含指定目录下的指定类型的文件

```xml
<build>
	<resources>
        <!-- 指定包含 src/main/java 目录下的所有 .xml/.properties 文件 -->
		<resource>
			<directory>src/main/java</directory>
			<includes>
				<include>**/*.xml</include>
                <include>**/*.properties</include>
			</includes>
		</resource>
	</resources>
</build>
```

#### 方案3、忽略指定目录下的文件（推荐）

```xml
<build>
	<resources>
        <!-- 指定包含 src/main/java 目录下的所有不是以 .java 结尾的文件 -->
		<resource>
			<directory>src/main/java</directory>
			<excludes>
				<exclude>**/*.java</exclude>
			</excludes>
		</resource>
	</resources>
</build>
```



### 包含 src/test/java 下的配置文件

不同于`src/main/java`，如果要包含`src/test/java`下的文件则应该将`<resources>`、`<resource>`标签替换为`testResources`、`testResource`。`<includes>` `<excludes>`用法和`main`目录一致。如下：

```xml
<build>
	<testResources>
		<testResource>
			<directory>src/test/java</directory>
			<excludes>
				<exclude>**/*.java</exclude>
			</excludes>
		</testResource>
	</testResources>
</build>
```


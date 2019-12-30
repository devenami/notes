## 使用java调用maven

### 依赖

```xml
<dependency>
    <groupId>org.apache.maven.shared</groupId>
    <artifactId>maven-invoker</artifactId>
    <version>3.0.1</version>
</dependency>
```

### 代码

```java
	 // 调用请求
      InvocationRequest request = new DefaultInvocationRequest();
      request.setPomFile(new File("path to pom.xml");
      List<String> command = new ArrayList<>();
      command.add("clean");
      command.add("package");
      request.setGoals(command);

      // 开始调用
      Invoker invoker = new DefaultInvoker();
      invoker.setMavenHome(new File("path to maven home"));
      InvocationResult result = invoker.execute(request);

      // 更新状态为执行中
      updateState(MavenState.EXECUTING);

      int resultCode = result.getExitCode();

      // 更新成功或失败状态
      if (resultCode == 0) {
          // 执行成功
      }
```

**获取maven执行输出**

```java
// 通过为 invoker 添加输出处理器监听maven产生的日志
private void addMavenOutputHandler(Invoker invoker) {
    invoker.setOutputHandler(line -> System.out::println);
}
```


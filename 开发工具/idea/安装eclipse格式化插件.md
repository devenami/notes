### 前言

在团队开发中，为了避免冲突，我们一般会采用统一的代码格式化工具，也会编写自己的`code-style`文件。如果团队中主力使用的是`eclipse`，并且`code-style`文件只有`eclipse`仅可使用的，那对于我们喜欢用`idea`的程序员来说，这种限制简直是灾难性的，试想一下，如果我们的代码格式化方式和同时的代码格式化方式不一样，那么当我们同事操作一个文件的时候，并且同时进行代码格式化与代码提交，那么提交后的代码冲突数量可想而知。为了解决这种问题，并兼容同事们使用`eclipse`编写的代码，我们就需要在`idea`中使用`eclipse`的`code-style`文件

### 插件介绍

`Eclipse Code Formate`插件是一个可以是用户在`idea`中使用`eclipse`的`code-style`文件的工具。

### 安装

通过`idea`的插件仓库搜索`Eclipse Code Formate`进行安装，安装完成后重启`idea`

### 配置

1. 选择`settings`-`Other Settings`选择`Eclipse Code Formate`
2. 启用`Use the Eclipse formatter`
3. `Eclipse Java Formatter config file`配置为`eclipse`的`code-style`文件在本地的路径
4. `Import order`启动`From file`，并配置为`code-style`文件的本地路径

### 使用及建议

* 使用`Ctrl`+`Alt`+`L`格式化代码后，`Event Log`会出现`*.java formatted successfully by Eclipse code formatter`，说明插件设置成功。
* 尽量格式化资源的代码，不要全局格式化。
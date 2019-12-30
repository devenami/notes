## 问题

同事在**pom**文件中新增了一条依赖项，但是将代码克隆到本地后使用**idea**无法下载该**jar**包。提示*offline mode*等等

## 原因

根据提示可知，当前maven开启了离线模式：仅使用本地仓库中现有的依赖项。

## 解决方案

`File -> Settings -> Build,Execution,Deployment -> Maven`关闭`Work offline`即可解决。
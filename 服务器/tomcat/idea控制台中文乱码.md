## 原因

maven在idea中使用系统控制台，windows系统的默认编码不是UTF-8，而tomcat的默认控制台编码为UTF-8，从而导致

## 解决方案

找到`tomcat`的根目录，`conf`-> `logging.properties`

注释`java.util.logging.ConsoleHandler.encoding = UTF-8`或者将`UTF-8`修改为`GBK`


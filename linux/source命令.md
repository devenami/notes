## 定义

source命令也称为“点命令”，也就是一个点符号（.）,是bash的内部命令。

## 功能

使Shell读入指定的Shell程序文件并依次执行文件中的所有语句 source命令通常用于重新执行刚修改的初始化文件，使之立即生效，而不必注销并重新登录。

## 用法

`source filename` 或` . filename`

 source命令(从 C Shell 而来)是bash shell的内置命令;点命令(.)，就是个点符号(从Bourne Shell而来)是source的另一名称。



## source filename 与 sh filename 及./filename执行脚本的区别在那里呢？

1.当shell脚本具有可执行权限时，用sh filename与./filename执行脚本是没有区别得。./filename是因为当前目录没有在PATH中，所有"."是用来表示当前目录的。

2.sh filename 重新建立一个子shell，在子shell中执行脚本里面的语句，该子shell继承父shell的环境变量，但子shell新建的、改变的变量不会被带回父shell，除非使用export。

3.source filename：这个命令其实只是简单地读取脚本里面的语句依次在当前shell里面执行，没有建立新的子shell。那么脚本里面所有新建、改变变量的语句都会保存在当前shell里面。  



## 引用

[source命令](https://www.cnblogs.com/wqbin/p/10244063.html)
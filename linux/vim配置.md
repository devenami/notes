# VIM 配置

## VIM中配置TAB键现实为空格

### 可修改的文件

可以修改的文件有两处：

全局配置：

```shell
vim /etc/vim/vimrc
```

个人配置(推荐)：

```shell
cp /usr/share/vim/vimrc ~/.vimrc
vim ~/.vimrc
```

### vimrc中修改参数

```shell
set ts=4
set expandtab
set autoindent
```

### 修改已存在文件中的配置

Tab替换为空格

```shell
:set ts=4
:set expandtab
:%retab!
```

空格替换为Tab

```shell
:set ts=4
:set noexpandtab
:%retab!
```

`!`用于强制修改文件中所有的tab

## 默认显示行号

修改`~/.vimrc`增加如下内容

```shell
set nu
```

## 显示颜色

修改`~/.vimrc`增加如下内容

```shell
syntax enable
```


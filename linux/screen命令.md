## screen命令

### 创建会话

```shell
screen -S 会话名称
或者
screen -R 会话名称
```

### 查看会话列表

```shell
screen -ls
```

### 恢复会话

```shell
screen -r 会话名称
或
screen -R 会话名称
```

**恢复失败时：**

```shell
screen -d 会话名称
screen -r 会话名称
```

### 删除会话

```shell
screen -S 会话名称 -X quit
```


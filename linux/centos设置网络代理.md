## CentOS配置网络代理

### 涉及的环境变量

| 环境变量    | 描述                                                         | 值示例                                                       |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| http_proxy  | 为http变量设置代理；默认不填开头以http协议传输               | 10.0.0.51:8080 user:pass@10.0.0.10:8080 socks4://10.0.0.51:1080 socks5://192.168.1.1:1080 |
| https_proxy | 为https变量设置代理；                                        | 同上                                                         |
| ftp_proxy   | 为ftp变量设置代理；                                          | 同上                                                         |
| all_proxy   | 全部变量设置代理，设置了这个时候上面的不用设置               | 同上                                                         |
| no_proxy    | 无需代理的主机或域名； 可以使用通配符； 多个时使用“,”号分隔； | \*.aiezu.com,10.\*.\*.\*,192.168.\*.\*, *.local,localhost,127.0.0.1 |

### 修改文件

`/etc/profile`

```shell
export proxy="http://ip:1080" # 这里修改为指定协议的格式
export http_proxy=$proxy
export https_proxy=$proxy
export ftp_proxy=$proxy
export no_proxy="localhost, 127.0.0.1, ::1"
```


### 准备

#### 下载

下载地址`https://www.consul.io/downloads.html`

#### 安装

- 解压`unzip consule-consul_1.2.2_linux_amd64.zip`
- 拷贝 `cp ./consule /usr/bin`

### 命令（cli）

#### 帮助

`consul command -h`

#### API选项

- `-ca-file=<value>`- 与Consul通信时用于TLS的CA文件的路径。这也可以通过`CONSUL_CACERT`环境变量指定。
- `-ca-path=<value>` - 与Consul通信时用于TLS的CA证书目录的路径。这也可以通过`CONSUL_CAPATH`环境变量指定。
- `-client-cert=<value>`- verify_incoming启用时用于TLS的客户端证书文件的路径 。这也可以通过CONSUL_CLIENT_CERT 环境变量指定。
- `-client-key=<value>`- verify_incoming启用时用于TLS的客户机密钥文件的路径 。这也可以通过CONSUL_CLIENT_KEY 环境变量指定。
- `-http-addr=<addr>` - 具有端口的Consul代理的地址。这可以是IP地址或DNS地址，但必须包含端口。这也可以通过`CONSUL_HTTP_ADDR`环境变量指定。在Consul 0.8及更高版本中，默认值为[http://127.0.0.1:8500](http://127.0.0.1:8500/)，可以选择使用https。也可以通过设置环境变量将方案设置为HTTPS `CONSUL_HTTP_SSL=true`。
- `-tls-server-name=<value>` - 通过TLS连接时用作SNI主机的服务器名称。这也可以通过`CONSUL_TLS_SERVER_NAME` 环境变量指定。
- `-token=<value>`- 在请求中使用的ACL令牌。这也可以通过`CONSUL_HTTP_TOKEN`环境变量指定。如果未指定，则查询将默认为HTTP地址处的Consul代理的令牌。
- `-datacenter=<name>` - 要查询的数据中心的名称。如果未指定，则查询将默认为HTTP地址处的Consul代理的数据中心。

- `-stale` - 允许任何Consul服务器（非领导者）响应此请求。这允许更低的延迟和更高的吞吐量，但可能导致过时的数据。此选项对非读取操作没有影响。默认值为false。

#### 代理人(agent)

该`consul agent`命令是Consul的核心：它运行代理，执行维护成员资格信息，运行检查，公布服务，处理查询等重要任务。 

`console agent -dev`

#### 目录(category)

| 命令                                     | 解释                       |
| ---------------------------------------- | -------------------------- |
| consul category datacenters              | 列出多有的数据中心         |
| consul category nodes                    | 列出所有的节点             |
| console category nodes -service=redis    | 列出提供特定服务的所有节点 |
| consul category services                 | 列出所有服务               |
| consul category services -node=worker-01 | 列出节点上的所有服务       |

##### 数据中心

用法： `consul catalog datacenters [options]`

##### 节点

用法： `consul catalog nodes [options]` 

目录列表节点选项

- `-detailed`- 输出有关节点的详细信息，包括其地址和元数据。
- `-near=<string>` - 节点名称，用于根据从该节点估计的往返时间按升序对节点列表进行排序。传递`"_agent"`将使用此代理的节点进行排序。
- `-node-meta=<key=value>` - 使用给定键=值对过滤节点的元数据。可以多次指定该标志以过滤多个元数据源。
- `-service=<id or name>` - 过滤节点的服务ID或名称。仅返回提供给定服务的节点。

##### 服务

用法： `consul catalog services [options]` 

目录列表节点选项

- `-node=<id or name>`- `id or name`列出服务的节点。
- `-node-meta=<key=value>`- 使用给定`key=value`对过滤节点的元数据 。如果指定，则仅返回在与给定元数据匹配的节点上运行的服务。可以多次指定该标志以过滤多个元数据源。
- `-tags` - 将每个服务的标签显示为每个服务条目旁边的逗号分隔列表。

#### 链接(connect)

##### 证书颁发机构(CA)

**get-config**

此命令显示当前的CA配置。 

用法： `consul connect ca get-config [options]` 

**set-config**

修改当前的CA配置。如果这导致使用新的根证书，则将触发[根循环](https://www.consul.io/docs/connect/ca.html#root-certificate-rotation)过程。

用法： `consul connect ca set-config [options]`

命令选项

- `-config-file`- （必需）指定用于新配置的JSON格式的文件。

##### 代理（proxy）

用法： `consul connect proxy [options]` 

代理选项

- `-service` - 此代理表示的服务的名称。此服务不需要实际存在于Consul目录中，但需要正确的ACL权限（`service:write`）。
- `-upstream`- 支持连接的上游服务。格式应为'name：addr'，例如'db：8181'。这将使端口8181上的“db”可用。当与端口8181建立常规TCP连接时，代理将服务发现“db”并建立一个标识为`-service`值的Connect mTLS连接。该标志可以重复多次。
- `-listen` - 用于侦听代理服务的入站连接的地址。必须使用-service和-service-addr指定。如果未指定，则不启动入站侦听器。
- `-service-addr` - 代理的本地服务的地址。必需的 `-listen`。
- `-register` - 向当地Consul代理自行注册，使该代理在目录中作为Connect-capable服务提供。这只适用于`-listen`。
- `-register-id`- `-register`设置为消除服务ID歧义时服务的可选ID后缀。默认情况下，服务ID为“-proxy”，其中`<service>`是`-service`的值。
- `-log-level`- 指定日志级别。
- `-pprof-addr`- 通过pprof启用调试。提供host：port（或只是'：port'）可以在该地址上分析HTTP端点。
- `-proxy-id` - 本地代理上的代理ID。这仅适用于测试托管代理模式。

例子

下面的示例显示如何启动本地代理以建立表示前端服务的“db”的出站连接。一旦运行，任何创建到指定端口（8181）的TCP连接的进程将建立与标识为“前端”的“db”的相互TLS连接。

```
$ consul connect proxy -service frontend -upstream db:8181
```

下一个示例启动一个本地代理，该代理也接受端口8443上的入站连接，授权连接，然后将其代理到端口8080：

```
$ consul connect proxy \
    -service frontend \
    -service-addr 127.0.0.1:8080 \
    -listen ':8443'
```

#### 加入(join)

该`join`命令告知Consul代理加入现有集群。新的Consul代理必须与群集中的至少一个现有成员连接才能加入现有群集。在加入该一个成员之后，八卦层接管，在集群中传播更新的成员资格状态。

如果您未加入现有群集，则该代理程序是其自己的隔离群集的一部分。其他节点可以加入它。

代理可以多次加入其他代理而不会出现问题。如果已经是集群一部分的节点加入另一个节点，则两个节点的集群将成为一个集群。

用法： `consul join [options] address ...` 

命令选项

- `-wan` - 对于以服务器模式运行的代理，代理将尝试加入WAN群集中的其他服务器集群。这用于在多个数据中心之间形成桥梁。

#### KV

该`kv`命令用于通过命令行与Consul的KV存储进行交互。它公开了用于从商店插入，更新，读取和删除的顶级命令。 

用法： `consul kv <subcommand>` 

##### 删除（delete）

用法： `consul kv delete [options] KEY_OR_PREFIX` 

KV删除选项

- `-cas` - 执行检查和设置操作。指定此值还需要设置-modify-index标志。默认值为false。
- `-modify-index=<int>` - 表示密钥的ModifyIndex的无符号整数。这与-cas标志结合使用。
- `-recurse` - 递归删除路径中的所有键。默认值为false。

例子

要删除KV存储中名为“redis / config / connections”的键的值，请执行以下操作： 

`consul kv delete redis/config/connections`

如果密钥不存在，则命令不会出错，并返回成功消息

要仅删除自给定索引后尚未修改的密钥，请指定`-cas`和`-modify-index`标记：

```
$ consul kv get -detailed redis/config/connections | grep ModifyIndex
ModifyIndex      456

$ consul kv delete -cas -modify-index=123 redis/config/connections
Error! Did not delete key redis/config/connections: CAS failed

$ consul kv delete -cas -modify-index=456 redis/config/connections
Success! Deleted key: redis/config/connections
```

要以递归方式删除以给定前缀开头的所有键，请指定 `-recurse`标志：

```
$ consul kv delete -recurse redis/
Success! Deleted keys with prefix: redis/
```

**尾随斜杠**在递归删除操作中**很重要**，因为Consul会对提供的前缀执行贪婪匹配。如果你使用“foo”作为键，这将递归删除任何以这些字母开头的键，例如“foo”，“food”和“football”，而不仅仅是“foo”。要确保删除文件夹，请始终使用尾部斜杠。

将`-cas`选项组合在一起无效`-recurse`，因为您在单个操作中删除前缀下的多个键：

```
$ consul kv delete -cas -recurse redis/
Cannot specify both -cas and -recurse!
```

##### 导出（export）

该`kv export`命令用于从Consul的KV存储中检索给定前缀的KV对，并将JSON表示写入stdout。这可以与命令“consul kv import”一起使用，以在Consul集群之间移动整个树。 

用法： `consul kv export [PREFIX]` 

例子

要在键值存储中的“vault /”处导出树：

```
$ consul kv export vault/
# JSON output
```

##### 取值（get）

用法： `consul kv get [options] [KEY_OR_PREFIX]` 

KV获取选项

- `-base64` - Base 64对值进行编码。默认值为false。
- `-detailed` - 除了值之外，还提供有关密钥的其他元数据，例如ModifyIndex和可能已在密钥上设置的任何标志。默认值为false。
- `-keys` - 列出以给定前缀开头但不是其值的键。如果您只需要键名本身，这将特别有用。此选项通常与-separator选项结合使用。默认值为false。
- `-recurse` - 递归查看前缀为给定路径的所有键。默认值为false。
- `-separator=<string>` - 用作键之间分隔符的字符串。默认值为“/”，但只有在与-keys标志配对时才考虑此选项。

例子

要仅列出以指定前缀开头的键，请使用“-keys”选项。这样性能更高，并且有效载荷更小：

```
$ consul kv get -keys redis/config/
redis/config/connections
redis/config/cpu
redis/config/memory
```

默认情况下，该`-keys`操作使用“/”分隔符，这意味着它不会递归到该分隔符之外。您可以通过设置选择不同的分隔符 `-separator="<string>"`。

```
$ consul kv get -keys -separator="c" redis
redis/c
```

或者，您可以通过将分隔符设置为空字符串来完全禁用分隔符：

```
$ consul kv get -keys -separator="" redis
redis/config/connections
redis/config/cpu
redis/config/memory
```

要列出根目录下的所有密钥，只需省略prefix参数：

```
$ consul kv get -keys
memcached/
redis/
```

##### 导入(import)

该`kv import`命令用于从`kv export`命令生成的JSON表示中导入KV对。 

用法： `consul kv import [DATA]` 

例子

要从文件导入，请在文件名前加上`@`：

```
$ consul kv import @values.json
# Output
```

要从stdin导入，请`-`用作数据参数：

```
$ cat values.json | consul kv import -
# Output
```

您也可以直接传递JSON，但必须小心shell转义：

```
$ consul kv import "$(cat values.json)"
# Output
```

##### 存值 （put）

该`kv put`命令将数据写入KV存储中的给定路径。 

用法： `consul kv put [options] KEY [DATA]` 

选项

- `-acquire` - 获得钥匙锁。如果密钥不存在，则此操作将创建密钥并获取锁定。会话必须已存在并通过-session标志指定。默认值为false。
- `-base64` - 将数据视为基数为64的编码。默认值为false。
- `-cas` - 执行检查和设置操作。指定此值还需要设置-modify-index标志。默认值为false。
- `-flags=<int>`- 要分配给此KV对的无符号整数值。Consul不会读取此值，因此客户端可以使用此值但对其用例有意义。默认值为0（无标志）。
- `-modify-index=<int>` - 表示密钥的ModifyIndex的无符号整数。这与-cas标志结合使用。
- `-release`- 放弃给定路径上钥匙的锁定。这需要设置-session标志。密钥必须由会话保存才能解锁。默认值为false。
- `-session=<string>` - 用户定义的此会话标识符为字符串。这通常与-acquire和-release操作一起使用来构建强大的锁定，但它可以在任何键上设置。默认值为空（无会话）。

例子

对于更长或更敏感的值，可以通过在`@`符号前面添加前缀来读取文件：

```
$ consul kv put redis/config/password @password.txt
Success! Data written to: redis/config/connections
```

或者通过指定`-`符号从stdin读取值：

```
$ echo "5" | consul kv put redis/config/password -
Success! Data written to: redis/config/connections

$ consul kv put redis/config/password -
5
<CTRL+D>
Success! Data written to: redis/config/connections
```

要创建或调整锁定，请使用`-acquire`和`-session`标志。会话必须已存在（此命令不会创建或管理它）：

```
$ consul kv put -acquire -session=abc123 redis/lock/update
Success! Lock acquired on: redis/lock/update
```

完成后，释放锁：

```
$ consul kv put -release -session=acb123 redis/lock/update
Success! Lock released on: redis/lock/update
```

### 代理(agent)

使用该`consul agent`命令启动代理程序。此命令阻止，永久运行或直到被告知退出。agent命令有很多种`configuration options`，但大多数都有合理的默认值。

运行时`consul agent`，您应该看到与此类似的输出：

```
$ consul agent -data-dir=/tmp/consul
==> Starting Consul agent...
==> Consul agent running!
       Node name: 'Armons-MacBook-Air'
      Datacenter: 'dc1'
          Server: false (bootstrap: false)
     Client Addr: 127.0.0.1 (HTTP: 8500, DNS: 8600)
    Cluster Addr: 192.168.1.43 (LAN: 8301, WAN: 8302)

==> Log data will now stream in as it occurs:

    [INFO] serf: EventMemberJoin: Armons-MacBook-Air.local 192.168.1.43
...
```

`consul agent`输出有几条重要信息：

- **节点名称**：这是代理的唯一名称。默认情况下，这是计算机的主机名，但您可以使用该`-node`标志对其进行自定义 。
- **数据中心**：这是代理配置为运行的数据中心。Consul为多个数据中心提供一流的支持; 但是，要高效工作，必须将每个节点配置为报告其数据中心。该`-datacenter`标志可用于设置数据中心。对于单DC配置，代理将默认为“dc1”。
- **服务器**：这表示代理是在服务器还是客户端模式下运行。服务器节点具有参与共识仲裁，存储群集状态和处理查询的额外负担。另外，服务器可以处于“引导”模式。多个服务器无法处于引导模式，因为这会使群集处于不一致状态。
- **客户端地址**：这是用于代理的客户端接口的地址。这包括HTTP和DNS接口的端口。默认情况下，这仅绑定到localhost。如果更改此地址或端口，则必须`-http-addr` 在运行命令时指定，`consul members`以指示如何联系代理。其他应用程序也可以使用HTTP地址和端口 来控制Consul。
- **Cluster Addr**：这是用于群集中Consul代理之间通信的端口和端口集。并非群集中的所有Consul代理都必须使用相同的端口，但所有其他节点都**必须**可以访问此地址。

当在运行`systemd`在Linux上，领事通知发送systemd `READY=1`到`$NOTIFY_SOCKET`当LAN连接已经完成。为此，必须设置`join`或`retry_join`选项，并且必须设置服务定义文件`Type=notify`。

#### 命令行配置

- `-advertise` - 广告地址用于将我们通告的地址更改为群集中的其他节点。默认情况下，`-bind`通告地址。但是，在某些情况下，可能存在无法绑定的可路由地址。此标志允许闲聊不同的地址以支持此功能。如果此地址不可路由，则该节点将处于恒定的振荡状态，因为其他节点将不可路由性视为故障。
- `-advertise-wan` - 广告WAN地址用于将我们通告的地址更改为通过WAN加入的服务器节点。当与`translate_wan_addrs`配置选项结合使用时，也可以在客户端代理上设置此选项。默认情况下，`-advertise`通告地址。但是，在某些情况下，所有数据中心的所有成员都不能位于同一物理或虚拟网络上，尤其是混合云和私有数据中心的混合设置。此标志使服务器节点通过公共网络为WAN进行闲聊，同时使用专用VLAN互相闲聊及其客户端代理，并且如果远程数据中心是远程数据中心，则允许从远程数据中心访问此地址时访问客户端代理。配置了`translate_wan_addrs`。
- `-bootstrap` - 此标志用于控制服务器是否处于“引导”模式。重要的是，在此模式下，*每个*数据中心只能运行一台服务器。从技术上讲，允许自举模式的服务器作为Raft领导者自行选举。重要的是只有一个节点处于这种模式; 否则，无法保证一致性，因为多个节点能够自我选择。在引导群集后，建议不要使用此标志。
- `-bootstrap-expect` - 此标志提供数据中心中预期的服务器数。不应提供此值，或者该值必须与群集中的其他服务器一致。提供后，Consul将等待指定数量的服务器可用，然后引导群集。这允许自动选择初始领导者。这不能与传统`-bootstrap`标志一起使用。此标志需要`-server`模式。
- `-bind` - 应绑定到内部群集通信的地址。这是群集中所有其他节点都应该可以访问的IP地址。默认情况下，这是“0.0.0.0”，这意味着Consul将绑定到本地计算机上的所有地址，并将 第一个可用的私有IPv4地址通告给群集的其余部分。如果有**多个私有IPv4地址**可用，Consul将在启动时退出并显示错误。如果指定“[::]”，Consul将 通告第一个可用的公共IPv6地址。如果有**多个**可用的**公共IPv6地址**，Consul将在启动时退出并显示错误。Consul同时使用TCP和UDP以及相同的端口。如果您有防火墙，请务必同时允许这两种协议。
- `-serf-wan-bind` - 应该绑定到Serf WAN八卦通信的地址。默认情况下，该值遵循与`-bind`相同的规则，如果未指定，`-bind`则使用该选项。
- `-serf-lan-bind` - 应该绑定到Serf LAN八卦通信的地址。这是群集中所有其他LAN节点都应该可以访问的IP地址。默认情况下，该值遵循与`-bind`命令行标志相同的规则，如果未指定，`-bind`则使用该选项。
- `-client`- Consul将绑定客户端接口的地址，包括HTTP和DNS服务器。默认情况下，这是“127.0.0.1”，仅允许环回连接。
- `-config-file` - 要加载的配置文件。有关此文件格式的更多信息，请阅读“ 配置文件”部分。可以多次指定此选项以加载多个配置文件。如果多次指定，则稍后加载的配置文件将与先前加载的配置文件合并。在配置合并期间，单值键（string，int，bool）将简单地替换它们的值，而列表类型将被附加在一起。
- `-config-dir` - 要加载的配置文件的目录。Consul将使用后缀“.json”或“.hcl”加载此目录中的所有文件。加载顺序是按字母顺序排列的，并且使用与上述`config-file`选项相同的合并例程 。可以多次指定此选项以加载多个目录。未加载config目录的子目录。有关配置文件格式的详细信息，请参阅“配置文件”部分。
- `-config-format`- 要加载的配置文件的格式。通常，Consul会从“.json”或“.hcl”扩展名中检测配置文件的格式。将此选项设置为“json”或“hcl”会强制Consul解释具有或不具有扩展名的任何文件，以便以该格式进行解释。
- `-data-dir` - 此标志为代理程序存储状态提供数据目录。这是所有代理商都需要的。该目录在重新启动后应该是持久的。这对于在服务器模式下运行的代理尤其重要，因为它们必须能够持久化群集状态。此外，该目录必须支持使用文件系统锁定，这意味着某些类型的已安装文件夹（例如VirtualBox共享文件夹）可能不适合。**注意：**服务器代理和非服务器代理都可以在此目录中的状态中存储ACL令牌，因此读访问可以授予对服务器上的任何令牌以及非服务器上的服务注册期间使用的任何令牌的访问权限。在基于Unix的平台上，文件使用0600权限编写，因此您应确保只有受信任的进程才能与Consul作为同一用户执行。在Windows上，您应确保该目录具有适当的权限，因为这些权限将被继承。
- `-datacenter` - 此标志控制代理程序运行的数据中心。如果未提供，则默认为“dc1”。Consul拥有对多个数据中心的一流支持，但它依赖于正确的配置。同一数据中心中的节点应位于单个LAN上。
- `-dev`- 启用开发服务器模式。这对于快速启动Consul代理并关闭所有持久性选项非常有用，可以启用内存服务器，该服务器可用于快速原型设计或针对API进行开发。在此模式下， Connect已启用，默认情况下将在启动时创建新的根CA证书。此模式**不适**用于生产用途，因为它不会将任何数据写入磁盘。
- `-disable-host-node-id` - 将此设置为true将阻止Consul使用来自主机的信息生成确定性节点ID，而是生成将保留在数据目录中的随机节点ID。在同一主机上运行多个Consul代理进行测试时，这非常有用。在版本0.8.5之前的Consul中默认为false，在0.8.5及更高版本中默认为true，因此您必须选择加入基于主机的ID。使用<https://github.com/shirou/gopsutil/tree/master/host>生成基于主机的ID ，这是与HashiCorp的[Nomad](https://www.nomadproject.io/)共享的 ，因此如果您选择使用基于主机的ID，那么Consul和Nomad将使用信息在主机上自动在两个系统中分配相同的ID。
- `-disable-keyring-file` - 如果设置，密钥环将不会持久保存到文件中。关机时任何已安装的密钥都将丢失，`-encrypt`启动时只有给定的 密钥可用。默认为false。
- `-dns-port` - 要侦听的DNS端口。这将覆盖默认端口8600.这在Consul 0.7及更高版本中可用。
- `-domain` - 默认情况下，Consul在“consul”中响应DNS查询。域。此标志可用于更改该域。假定此域中的所有查询都由Consul处理，不会以递归方式解析。
- `-enable-script-checks`这可以控制是否在此代理上启用了执行脚本的运行状况检查，默认为`false`运营商必须选择允许这些检查。如果启用，建议还启用ACL以控制允许哪些用户注册新检查以执行脚本。这是在Consul 0.9.0中添加的。
- `-encrypt` - 指定用于加密Consul网络流量的密钥。该密钥必须是16字节的Base64编码。创建加密密钥的最简单方法是使用 `consul keygen`。群集中的所有节点必须共享相同的加密密钥才能进行通信。提供的密钥将自动持久保存到数据目录，并在重新启动代理时自动加载。这意味着要加密Consul的八卦协议，只需在每个代理的初始启动序列上提供一次该选项。如果在使用加密密钥初始化Consul之后提供，则忽略提供的密钥并显示警告。
- `-hcl` - HCL配置片段。此HCL配置片段将附加到配置中，并允许在命令行上指定配置文件的所有选项。可以多次指定此选项。这是在Consul 1.0中添加的。
- `-http-port`- 要监听的HTTP API端口。这将覆盖默认端口8500.将Consul部署到通过环境传输HTTP端口的环境（例如CloudFoundry等PaaS）时，此选项非常有用，允许您通过Procfile直接设置端口。
- `-join` - 启动时加入的另一个代理的地址。可以多次指定，以指定要加入的多个代理。如果Consul无法加入任何指定的地址，则代理启动将失败。默认情况下，代理在启动时不会加入任何节点。请注意，`retry_join`在自动执行Consul群集部署时，使用 可能更适合帮助缓解节点启动竞争条件。

- `-retry-join`- 类似`-join`但允许在第一次尝试失败时重试连接。这对于您知道地址最终可用的情况很有用。该列表可以包含IPv4，IPv6或DNS地址。如果Consul在非默认的Serf LAN端口上运行，则必须同时指定。IPv6必须使用“括号”语法。如果给出了多个值，则会按列出的顺序尝试和重试它们，直到第一个成功为止。这里有些例子：

  ```
  # Using a DNS entry
  $ consul agent -retry-join "consul.domain.internal"
  ```

  ```
  # Using IPv4
  $ consul agent -retry-join "10.0.4.67"
  ```

  ```
  # Using IPv6
  $ consul agent -retry-join "[::1]:8301"
  ```

#### 配置文件

示例配置文件

```
{
  "datacenter": "east-aws",
  "data_dir": "/opt/consul",
  "log_level": "INFO",
  "node_name": "foobar",
  "server": true,
  "watches": [
    {
        "type": "checks",
        "handler": "/usr/bin/health-check-handler.sh"
    }
  ],
  "telemetry": {
     "statsite_address": "127.0.0.1:2180"
  }
}
```

示例配置文件，带有TLS

```
{
  "datacenter": "east-aws",
  "data_dir": "/opt/consul",
  "log_level": "INFO",
  "node_name": "foobar",
  "server": true,
  "addresses": {
    "https": "0.0.0.0"
  },
  "ports": {
    "https": 8080
  },
  "key_file": "/etc/pki/tls/private/my.key",
  "cert_file": "/etc/pki/tls/certs/my.crt",
  "ca_file": "/etc/pki/tls/certs/ca-bundle.crt"
}
```

特别参见`ports`设置的使用：

```
"ports": {
  "https": 8080
}
```

除非为`https`端口分配了端口号，否则Consul不会为HTTP API启用TLS `> 0`。

#### 服务定义

服务发现的主要目标之一是提供可用服务的目录。为此，代理提供了一种简单的服务定义格式来声明服务的可用性，并可能将其与运行状况检查相关联。如果运行状况检查与服务关联，则将其视为应用程序级别。服务在配置文件中定义，或在运行时通过HTTP接口添加。 

要配置服务，请将服务定义作为`-config-file`代理提供给代理，或将其置于`-config-dir`代理中。该文件必须以Consul加载的`.json`或`.hcl`扩展名结尾。可以通过向`SIGHUP`代理发送更新检查定义。或者，可以使用HTTP API动态注册服务。 

服务定义是一种如下所示的配置。此示例显示了所有可能的字段，但请注意，只需要几个字段。

```
{
  "service": {
    "name": "redis",
    "tags": ["primary"],
    "address": "",
    "meta": {
      "meta": "for my service"
    },
    "port": 8000,
    "enable_tag_override": false,
    "checks": [
      {
        "args": ["/usr/local/bin/check_redis.py"],
        "interval": "10s"
      }
    ],
    "kind": "connect-proxy",
    "proxy_destination": "redis",
    "connect": {
      "native": false,
      "proxy": {
        "command": [],
        "config": {}
      }
    }
  }
}
```

服务定义必须包括`name`和可任选地提供 `id`，`tags`，`address`，`port`，`check`，`meta`和`enable_tag_override`。如果没有提供，`id`则设置为`name`。要求所有服务每个节点都有唯一的ID，因此如果名称可能存在冲突，则应提供唯一ID。



可以使用`services`配置文件中的复数键一次提供多个服务定义 。

```
{
  "services": [
    {
      "id": "red0",
      "name": "redis",
      "tags": [
        "primary"
      ],
      "address": "",
      "port": 6000,
      "checks": [
        {
          "args": ["/bin/check_redis", "-p", "6000"],
          "interval": "5s",
          "ttl": "20s"
        }
      ]
    },
    {
      "id": "red1",
      "name": "redis",
      "tags": [
        "delayed",
        "secondary"
      ],
      "address": "",
      "port": 7000,
      "checks": [
        {
          "args": ["/bin/check_redis", "-p", "7000"],
          "interval": "30s",
          "ttl": "60s"
        }
      ]
    },
    ...
  ]
}
```



#### 检查定义

代理的主要角色之一是管理系统级和应用程序级运行状况检查。如果运行状况检查与服务关联，则将其视为应用程序级别。如果未与服务关联，则检查将监视整个节点的运行状况。

检查在配置文件中定义，或在运行时通过HTTP接口添加。通过HTTP接口创建的检查将与该节点保持一致。

脚本检查：

```
{
  "check": {
    "id": "mem-util",
    "name": "Memory utilization",
    "args": ["/usr/local/bin/check_mem.py", "-limit", "256MB"],
    "interval": "10s",
    "timeout": "1s"
  }
}
```

HTTP检查：

```
{
  "check": {
    "id": "api",
    "name": "HTTP API on port 5000",
    "http": "https://localhost:5000/health",
    "tls_skip_verify": false,
    "method": "POST",
    "header": {"x-foo":["bar", "baz"]},
    "interval": "10s",
    "timeout": "1s"
  }
}
```

本地服务的别名检查：

```
{
  "check": {
    "id": "web-alias",
    "alias_service": "web"
  }
}
```



### 集群搭建

服务端

```
consul agent -server -bootstrap-expect=1 \
    -data-dir=/tmp/consul -node=agent-one -bind=172.20.20.10 \
    -enable-script-checks=true -config-dir=/etc/consul.d
```

客户端

```
consul agent -data-dir=/tmp/consul -node=agent-two \
    -bind=172.20.20.11 -enable-script-checks=true -config-dir=/etc/consul.d
```

服务端调用 `join`将节点加入集群

```
consul join 172.20.20.11
```
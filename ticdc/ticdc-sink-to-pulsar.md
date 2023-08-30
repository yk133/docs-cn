---
title: 同步数据到 Pulsar
summary: 了解如何使用 TiCDC 将数据同步到 Pulsar。
---

# 同步数据到 Pulsar

本文介绍如何使用 TiCDC 创建一个将增量数据复制到 Pulsar 的 Changefeed。

## 创建同步任务，复制增量数据 Pulsar

使用以下命令来创建同步任务：

```shell
cdc cli changefeed create \
    --server=http://127.0.0.1:8300 \
--sink-uri="pulsar://127.0.0.1:6650/persistent://public/default/yktest?protocol=canal-json" \
--config=./t_changefeed.toml \
--changefeed-id="simple-replication-task"
```

```shell

todo ：创建成功的提示
```

- `--server`：TiCDC 集群中任意一个 TiCDC 服务器的地址。
- `--changefeed-id`：同步任务的 ID，格式需要符合正则表达式 `^[a-zA-Z0-9]+(\-[a-zA-Z0-9]+)*$`。如果不指定该 ID，TiCDC 会自动生成一个 UUID（version 4 格式）作为 ID。
- `--sink-uri`：同步任务下游的地址，详见：[Sink URI 配置 Pulsar](/ticdc/ticdc-sink-to-pulsar.md#sink-uri-配置-pulsar)。
- `--start-ts`：指定 changefeed 的开始 TSO。TiCDC 集群将从这个 TSO 开始拉取数据。默认为当前时间。
- `--target-ts`：指定 changefeed 的目标 TSO。TiCDC 集群拉取数据直到这个 TSO 停止。默认为空，即 TiCDC 不会自动停止。
- `--config`：指定 changefeed 配置文件，详见：[TiCDC Changefeed 配置参数](/ticdc/ticdc-changefeed-config.md)。

## Sink URI & Changefeed config 配置 `pulsar`

__Sink URI 用于指定 TiCDC 目标系统的连接信息，Changefeed config用来配置具体pulsar相关的参数。在实际使用过程中，SinkURI用来配置基本的连接信息，Changefeed config用来配置可控参数。__

Sink URI 遵循以下格式：

```shell
[scheme]://[userinfo@][host]:[port][/path]?[query_parameters]
```

一个通用的配置样例如下所示：

```shell
--sink-uri="pulsar://127.0.0.1:6650/persistent://abc/def/yktest?protocol=canal-json"

或者：

--sink-uri="pulsar://127.0.0.1:6650/yktest?protocol=canal-json"

```

URI 中可配置的的参数如下：

| 参数               | 描述                                                         |
| :------------------ | :------------------------------------------------------------ |
| `127.0.0.1`          | 下游 Pulsar 对外提供服务的 IP。                                 |
| `6650`               | 下游 Pulsar 的连接端口。                                          |
| `persistent://abc/def/yktest`           |  指定pulsar的租户/命名空间/topic 的全名。                                      |
| `yktest`    | 使用pulsar服务端的默认租户/默认命名空间的topic 的简写名。通常默认的租户是public，命名空间是default,yktest为具体的topic，相当于persistent://public/default/yktest |
 
Changefeed config参数：

```
[sink]
# 注意：当下游MQ为pulsar时，如果partition不为'ts','index-value','table','default'时，会自动透传你设置的字符串作为pulsar message中的key来路由消息。
# dispatchers = [
#    {matcher = ['test1.*', 'test2.*'], topic = "Topic 表达式 1", partition = "ts" },
#    {matcher = ['test3.*', 'test4.*'], topic = "Topic 表达式 2", partition = "index-value" },
#    {matcher = ['test1.*', 'test5.*'], topic = "Topic 表达式 3", partition = "table"},
#    {matcher = ['test6.*'], partition = "default"},
#    {matcher = ['test7.*'], partition = "test123"}
# ]

# protocol 用于指定编码消息时使用的格式协议
# 当下游类型是 Pulsar 时，仅支持canal-json、canal、maxwell
# protocol = "canal-json"

# 以下参数仅在下游为 Pulsar 时生效。
[sink.pulsar-config]
# Pulsar 使用Token进行pulsar服务端的认证
authentication-token = "xxxxxxxxxxxxx"
# Pulsar 使用Token进行pulsar服务端的认证，但这里是token的路径，会从ticdc server组建所在机器上读取
token-from-file="/data/pulsar/token-file.txt"
# Pulsar 使用basic帐号密码验证身份
basic-user-name="root"
# Pulsar  使用basic帐号密码验证身份
basic-password="password"
# Pulsar TLS加密认证证书路径
auth-tls-certificate-path="/data/pulsar/certificate"
# Pulsar TLS加密认证私钥路径
auth-tls-private-key-path="/data/pulsar/certificate.key"
# Pulsar TLS加密可信证书文件路径
tls-trust-certs-file-path="/data/pulsar/tls-trust-certs-file"
# Pulsar oauth2 issuer-url 更多详细配置请看pulsar官方介绍：https://pulsar.apache.org/docs/2.10.x/client-libraries-go/#oauth2-authentication
oauth2.oauth2-issuer-url="https://xxxx.auth0.com"
# Pulsar oauth2 audience
oauth2.oauth2-audience="https://xxxx.auth0.com/api/v2/"
# Pulsar oauth2 private-key
oauth2.oauth2-private-key="/data/pulsar/privateKey"
# Pulsar oauth2 client-id
oauth2.oauth2-client-id="0Xx...Yyxeny"
# Pulsar oauth2 oauth2-scope
oauth2.oauth2-scope="xxxx"

# Pulsar ticdc缓存pulsar producer的个数，默认10240个
pulsar-producer-cache-size=10240 
# Pulsar 数据压缩方式，默认不压缩，可选 lz4,zlib,zstd
compression-type= "lz4"
# Pulsar Pulsar客户端TCP建立链接时间，默认5秒
connection-timeout=5
# Pulsar 操作超时时间，默认30秒
operation-timeout=3
# Pulsar pulsar 一批发送消息最大数量，默认1000
batching-max-messages=1000
# Pulsar pulsar 批量发送消息等待时间，默认10毫秒
batching-max-publish-delay=10
# Pulsar pulsar 发送超时时间，默认30秒
send-timeout=30

```
 
 
### 最佳实践

* 由于Pulsar自动创建Topic，你至少需要在创建 changefeed 的时候设置 `protocol` 参数。 建议使用 `canal-json` 协议。
* `pulsar-producer-cache-size` 参数表示client会缓存producer的数量，因为pulsar 的producer只能对应一个topic,所以默认限制为10240个，采用lru方式缓存，最大约消耗内存1.1G。 


### TiCDC 使用 Pulsar 的认证与授权

使用 Pulsar 的 Token 认证时配置样例如下所示：

- Token

  Sink-URI
  ```shell
  --sink-uri="pulsar://127.0.0.1:6650/persistent://public/default/yktest?protocol=canal-json"
  ```
  config：
  ```shell
  [sink.pulsar-config]
  authentication-token = "xxxxxxxxxxxxx"
  ```

- Token from file

  Sink-URI
  ```shell
  --sink-uri="pulsar://127.0.0.1:6650/persistent://public/default/yktest?protocol=canal-json"
  ```
  config：
  ```shell
  [sink.pulsar-config]
  # Pulsar 使用Token进行pulsar服务端的认证，但这里是token文件的路径，会从ticdc server所在机器上读取
  token-from-file="/data/pulsar/token-file.txt"
  ```

- TLS 加密认证

  Sink-URI
  ```shell
  --sink-uri="pulsar+ssl://127.0.0.1:6650/persistent://public/default/yktest?protocol=canal-json"
  ```
  config：
  ```shell
  [sink.pulsar-config]
  # Pulsar TLS加密认证证书路径
  auth-tls-certificate-path="/data/pulsar/certificate"
  # Pulsar TLS加密认证私钥路径
  auth-tls-private-key-path="/data/pulsar/certificate.key"
  # Pulsar TLS加密可信证书文件路径
  tls-trust-certs-file-path="/data/pulsar/tls-trust-certs-file"
  ```
- OAuth2 认证

  Sink-URI
  ```shell
  --sink-uri="pulsar+ssl://127.0.0.1:6650/persistent://public/default/yktest?protocol=canal-json"
  ```
  config：
  ```shell
  [sink.pulsar-config]
  # Pulsar oauth2 issuer-url 更多详细配置请看pulsar官方介绍：https://pulsar.apache.org/docs/2.10.x/client-libraries-go/#oauth2-authentication
  oauth2.oauth2-issuer-url="https://xxxx.auth0.com"
  # Pulsar oauth2 audience
  oauth2.oauth2-audience="https://xxxx.auth0.com/api/v2/"
  # Pulsar oauth2 private-key
  oauth2.oauth2-private-key="/data/pulsar/privateKey"
  # Pulsar oauth2 client-id
  oauth2.oauth2-client-id="0Xx...Yyxeny"
  # Pulsar oauth2 oauth2-scope
  oauth2.oauth2-scope="xxxx"
  ```
 
 
## 自定义 Kafka Sink 的 Topic 和 Partition 的分发规则

### Matcher 匹配规则

以如下示例配置文件中的 `dispatchers` 配置项为例： 

```toml
[sink]
dispatchers = [
  {matcher = ['test1.*', 'test2.*'], topic = "Topic 表达式 1", partition = "ts" },
  {matcher = ['test3.*', 'test4.*'], topic = "Topic 表达式 2", partition = "index-value" },
  {matcher = ['test1.*', 'test5.*'], topic = "Topic 表达式 3", partition = "table"},
  {matcher = ['test6.*'], partition = "default"},
  {matcher = ['test7.*'], partition = "test123"}
]
```

- 对于匹配了 matcher 规则的表，按照对应的 topic 表达式指定的策略进行分发。例如表 test3.aa，按照 topic 表达式 2 分发；表 test5.aa，按照 topic 表达式 3 分发。
- 对于匹配了多个 matcher 规则的表，以靠前的 matcher 对应的 topic 表达式为准。例如表 test1.aa，按照 topic 表达式 1 分发。
- 对于没有匹配任何 matcher 的表，将对应的数据变更事件发送到 --sink-uri 中指定的默认 topic 中。例如表 test10.aa 发送到默认 topic。
- 对于匹配了 matcher 规则但是没有指定 topic 分发器的表，将对应的数据变更发送到 --sink-uri 中指定的默认 topic 中。例如表 test9.abc 发送到默认 topic。

### Topic 分发器

Topic 分发器用 topic = "xxx" 来指定，并使用 topic 表达式来实现灵活的 topic 分发策略。topic 的总数建议小于 1000。

Topic 表达式的基本规则为 `[prefix]{schema}[middle][{table}][suffix]`，详细解释如下：

- `prefix`：可选项，代表 Topic Name 的前缀。
- `{schema}`：可选项，用于匹配库名。
- `middle`：可选项，代表库表名之间的分隔符。
- `{table}`：可选项，用于匹配表名。
- `suffix`：可选项，代表 Topic Name 的后缀。

其中 `prefix`、`middle` 以及 `suffix` 仅允许出现大小写字母（`a-z`、`A-Z`）、数字（`0-9`）、点号（`.`）、下划线（`_`）和中划线（`-`）；`{schema}`、`{table}` 均为小写，诸如 `{Schema}` 以及 `{TABLE}` 这样的占位符是无效的。

一些示例如下：

- `matcher = ['test1.table1', 'test2.table2'], topic = "hello_{schema}_{table}"`
    - 对于表 `test1.table1` 对应的数据变更事件，发送到名为 `hello_test1_table1` 的 topic 中
    - 对于表 `test2.table2` 对应的数据变更事件，发送到名为 `hello_test2_table2` 的 topic 中
- `matcher = ['test3.*', 'test4.*'], topic = "hello_{schema}_world"`
    - 对于 `test3` 下的所有表对应的数据变更事件，发送到名为 `hello_test3_world` 的 topic 中
    - 对于 `test4` 下的所有表对应的数据变更事件，发送到名为 `hello_test4_ world` 的 topic 中
- `matcher = ['*.*'], topic = "{schema}_{table}"`
    - 对于 TiCDC 监听的所有表，按照“库名_表名”的规则分别分发到独立的 topic 中；例如对于 `test.account` 表，TiCDC 会将其数据变更日志分发到名为 `test_account` 的 Topic 中。

### DDL 事件的分发

#### 库级别 DDL

诸如 `create database`、`drop database` 这类和某一张具体的表无关的 DDL，称之为库级别 DDL。对于库级别 DDL 对应的事件，被发送到 `--sink-uri` 中指定的默认 topic 中。

#### 表级别 DDL

诸如 `alter table`、`create table` 这类和某一张具体的表相关的 DDL，称之为表级别 DDL。对于表级别 DDL 对应的事件，按照 dispatchers 的配置，被发送到相应的 topic 中。

例如，对于 `matcher = ['test.*'], topic = {schema}_{table}` 这样的 dispatchers 配置，DDL 事件分发情况如下：

- 若 DDL 事件中涉及单张表，则将 DDL 事件原样发送到相应的 topic 中。
    - 对于 DDL 事件 `drop table test.table1`，该事件会被发送到名为 `test_table1` 的 topic 中。
- 若 DDL 事件中涉及多张表（`rename table` / `drop table` / `drop view` 都可能涉及多张表），则将单个 DDL 事件拆分为多个发送到相应的 topic 中。
    - 对于 DDL 事件 `rename table test.table1 to test.table10, test.table2 to test.table20`，则将 `rename table test.table1 to test.table10` 的 DDL 事件发送到名为 `test_table1` 的 topic 中，将 `rename table test.table2 to test.table20` 的 DDL 事件发送到名为 `test.table2` 的 topic 中。

### Partition 分发器

在pulsar 中，一般习惯消费者连接该topic下所有partition来消费。
partition 分发器用 partition = "xxx" 来指定，支持 default、ts、index-value、table 四种 partition 分发器,但如果用户填入其他字段，则会在发送给pulsar server的消息中将该字段透传给Message的`key`，具体分发规则如下：

- default：有多个唯一索引（包括主键）时按照 table 模式分发；只有一个唯一索引（或主键）按照 index-value 模式分发；如果开启了 old value 特性，按照 table 分发
- ts：以行变更的 commitTs 做 Hash 计算并进行 event 分发
- index-value：以表的主键或者唯一索引的值做 Hash 计算并进行 event 分发
- table：以表的 schema 名和 table 名做 Hash 计算并进行 event 分发
- 其他：透传给Message的`key`，使用 pulsar 自身的分发机制。
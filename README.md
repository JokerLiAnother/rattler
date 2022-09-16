# Getting started

> 当前应用可监听`soft pak export`文件的生成路径，并将`export` 内容以`Json` 字符串的形式发送到消息队列中。监听`Import` 报关文档的消息队列，将报关文档内容以`xml` 文件形式保存在报关路径中，完成报关的**发送/接收** 功能，信息的传递均通过消息队列完成。同时还可启动**文件服务器** 以提供`api`访问下载`export` 文件和**税金单（PDF）** 。



## 命令介绍

直接运行`rattler` 查看命令选项

```shell
Rattler is used to communicate with SoftPak software to send or receive customs documents. For example:

Usage:
  rattler [command]

Available Commands:
  completion  Generate the autocompletion script for the specified shell
  help        Help about any command

Flags:
      --config string   config file (default is $HOME/.rattler/config.yaml)
  -h, --help            help for rattler

Use "rattler [command] --help" for more information about a command.

```

**.rattler.yaml**

```shell
# file server port
port: 7003

# Log path and level
log:
  level: debug
  directory: out/log/

# The saving path of tax bills and export files
ser-dir:
  nl:
    tax-bill:
    export:
  be:
    tax-bill:
    export:

# Import XML save directory
import:
  xml-dir:

# Export XML file listening path
watcher:
  nl:
    watch-dir:
    backup-dir:
  be:
    watch-dir:
    backup-dir:


# RabbitMQ configuration
rabbitmq:
  url: 'amqp://USER:PASSWORD@HOST:5672'
  # Import xml queue
  import:
    exchange: softpak.import.topic
    exchange-type: topic
    queue: softpak.import
  # Export xml queue
  export:
    exchange: softpak.export.topic
    exchange-type: topic
    # softpak.export.be / softpak.export.nl
    queue: softpak.export

```

<u>*==注意：密码中带有特殊字符时注意特殊符号的转义，使用**单引号**包裹包含特殊字符的字符串（如：‘amqp://user:@123!123@127.0.0.1:5672’）==*</u>



## API 接口说明

### Download tax bill PDF

下载税金单文件，用户指定访问源文件名和下载后的文件名进行文件下载。

#### Link：http://127.0.0.1:7003/download/pdf/:origin/:target?dc=nl

| 参数   | 必填 | 类型 | 说明                                             |
| ------ | ---- | ---- | ------------------------------------------------ |
| origin | y    | Str  | 需要下载的源文件名（***不包含文件后缀***）       |
| Target | y    | Str  | 下载后保存为指定的文件名（***不包含文件后缀***） |
| Dc     | y    | Str  | declare country (**NL\|BE**)                     |

***例如：http://localhost:7003/download/pdf/tax-bill/save_tax_bill?dc=nl***

### Download export XML

下载或访问`export` 结果文件

#### Link: http://127.0.0.1:7003/download/xml/:dc/:filename?download=1

| 参数     | 必填 | 类型 | 说明                                                         |
| -------- | ---- | ---- | ------------------------------------------------------------ |
| Dc       | y    | Str  | declare country (**NL\|BE**)                                 |
| Filename | y    | Str  | 文件名（***带文件后缀的完整文件名***）                       |
| Download | n    | Str  | 是否以下载的方式访问（***1｜0 ***），如果为1 则表示直接下载文件，其他值可在浏览器预览文件内容 |

***例如：http://localhost:7003/download/xml/be/17960_AI-2021-77635_18.xml?download=***



## 部署服务

### windows 部署

**以系统服务方式启动**

`windows` 非系统服务的命令不能直接使用`windows` 系统自带的命令创建系统服务。因此需要借助第三方工具来代替我们运行系统服务。

如: [winsw.exe](https://github.com/winsw/winsw/releases/tag/v2.11.0) ,下面以 `winsw.exe` 进行介绍:

- 首先将`winsw.exe` **重命名**（`exp:rattler-serve.exe`），创建`xml`配置文件**（配置文件名需要和执行文件名相同：`rattler-serve.xml`）**

```shell
<service>
    <id>RattlerService</id>
    <name>RattlerService</name>
    <description>This is a service that can be used to access SoftPak declaration documents</description>
    <executable>..\rattler.exe</executable>
    <arguments>--config ..\.rattler.yaml</arguments>
    <logmode>reset</logmode>
</service>

```

- 安装系统服务

```pow
rattler.exe install
```

- 其他命名

```postgresql
# 启动服务
rattler.exe start

# 停止服务
rattler.exe stop

# 卸载服务
rattler.exe uninstall
```



### Rattler 所有服务配置内容

`rattler` 打包文件中已经包含`winsw` 针对所有命令的配置，存放在路径`winsw/`中。文件结构如下：

```shell
➜  winsw ✗ tree
.
├── rattler.exe
├── rattler.xml

```


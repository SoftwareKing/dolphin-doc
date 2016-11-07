# Key 的设计

## key 的统一设计

> 注: 以下内容，不仅仅适用于服务注册，其他场合也是同样适用的。

### key 的层次结构

在考虑 key 的设计时，有一个重要背景需要考虑：

etcd3 的 KV **不支持目录层次结构**，所有的 key 都是在一个大的 KV 空间中。

这使得我们无法像在 zookeeper 或者 consul 中那样建立多层目录来对应实际的层次结构。

因此，我们不得已需要采用变通方案，原理时用类似层次结构的key来模拟：

```bash
/a/
/b/
/b/c
/b/d/e
```

例如上面的三个 key，模拟出来不同的虚拟目录("/a/"和"/b/")和虚拟文件("/b/c"和"/b/d/e")。

而在 etcd 真实的键值对空间中，只有虚拟文件对应的键值对实际存在，虚拟目录是没有对应的键值对的，虚拟目录只是我们用来帮助理解的一个虚假结构。

### key 遵循的规则

1. 所有 key 以 "/" 开头，表示从根目录开始
2. 所有表示虚拟目录的 key，以 "/" 结尾：如 "/a/" 表示根目录下的 a 子目录
3. 所有表示虚拟文件的 key，不以"/" 结尾: 如 "/a/b" 表示根目录下的 a 子目录中的 b 文件
4. 简单起见，不容许虚拟目录和虚拟文件重名，同一个路径只能是虚拟目录或者虚拟文件中的一个: 比如，不能同时有"/b/c"和"/b/c/"
5. 为了避免key重复，同一用途的所有 key 一定存放于根目录下的固定子目录：如 "/registry/" 用于服务注册

## 服务注册的 key 设计

### 根目录

服务注册的所有 key 都保存在虚拟目录 `/registry/` 下，因此所有的 key 都应该以 `/registry/` 开头：

```bash
/registry/
```

`/registry/` 作为整个服务注册的虚拟目录存在。

### 服务分组

每个服务分组对应到 `/registry/` 下的一个虚拟目录，目录名为分组名：

```bash
/registry/${group}/
```

以下为实际例子:

```bash
/registry/Dolphin/					// Dolphin 分组
/registry/Common/					// Common 分组
/registry/Infrastructure/			// Infrastructure 分组
```

每个服务分组下都有一个名为 `info` 的虚拟文件，用于保存当前服务分组的描述信息，具体值见下一节。

```bash
/registry/${group}/info
```

### 服务

每个服务对应到所属服务分组下的一个虚拟目录，目录名为服务名:

```bash
/registry/${group}/${service}/
```

以下为实际例子:

```bash
/registry/Common/VerifyCodeService/					// Common 分组下的验证码服务
/registry/Common/VerifyCodeService/					// Common 分组下的验证码服务
/registry/Common/AuthenticationService/				// Common 分组下的鉴权服务
```

每个服务下都有一个名为 `info` 的虚拟文件，用于保存当前服务的描述信息，具体值见下一节。

```bash
/registry/${group}/${service}/info
```

### 服务实例

每个服务实例对应到所属服务下的一个虚拟目录，目录名为"ip:port":

```bash
/registry/${group}/${service}/${ip}:${port}/
```

注意：这里的端口，指 gRPC 端口，不是 http 端口，也不是加密的 gRPC 端口。

以下为实际例子:

```bash
/registry/Common/VerifyCodeService/192.168.0.1:11080	// 运行于192.168.0.1:11080
/registry/Common/VerifyCodeService/192.168.0.1:21080	// 运行于192.168.0.1:21080
/registry/Common/VerifyCodeService/192.168.0.2:11080	// 运行于192.168.0.2:11080
```

注意，有可能在同一个IP地址的机器上运行两个甚至多个服务实例，因此必须通过IP加端口来区分。

对于每一个服务实例，我们有两个 key 来记录它的信息：

1. data： 用于保存服务实例的数据
2. status: 用于保存服务实例的状态
3. config: 用于保存服务实例的配置


#### 服务实例的数据

每个服务实例的数据保存在当前服务虚拟目录下，名为"data":

```bash
/registry/${group}/${service}/${ip}:${port}/data
```

#### 服务实例的状态

每个服务实例的状态保存在当前服务虚拟目录下，名为"status":

```bash
/registry/${group}/${service}/${ip}:${port}/status
```

#### 服务实例的配置

每个服务实例的配置保存在当前服务虚拟目录下，名为"config":

```bash
/registry/${group}/${service}/${ip}:${port}/config
```

具体每个key的value的设定请见下一节。


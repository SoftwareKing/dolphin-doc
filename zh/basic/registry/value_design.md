# Value的设计

## 整体设计

### 保存方案

考虑到平时 debug 时的方便性，存储在 etcd 键值对空间中的值，采用 json 格式。

这样必要时可以通过命令行工具 etcdctl 直接连接 etcd 查询。

## 详细设定

| key | 用途 |
|--------|--------|
|   /registry/${group}/info     |   服务分组的信息     |
|   /registry/${group}/${service}/info     |   服务的信息     |
|   /registry/${group}/${service}/${ip}:${port}/data     |   服务实例的数据     |
|   /registry/${group}/${service}/${ip}:${port}/status     |   服务实例的状态     |
|   /registry/${group}/${service}/${ip}:${port}/config     |   服务实例的配置     |

### 信息

#### 服务分组的信息

保存服务分组的信息，用于服务中心等页面展示，具体值包括：

| 名字 | 类型 | 描述 |
|--------|--------|--------|
|   displayName     |    string    |    显示名称，如果为空则显示为${group}    |
|   department     |    string    |    所属部门,服务中心页面支持根据部门过滤服务分组    |
|   description     |    string    |    服务分组的描述    |

#### 服务的信息

保存服务的信息，用于服务中心等页面展示，具体值包括：

| 名字 | 类型 | 描述 |
|--------|--------|--------|
|   displayName     |    string    |    显示名称    |
|   description     |    string    |    描述分组    |

### 服务实例

#### 服务实例的数据

服务实例的数据，用于服务中心等页面展示，

具体值包括：

| 名字 | 类型 | 描述 |
|--------|--------|--------|
|   serviceType     |    string    |    服务类型，dolphin框架必填    |
|   serviceVersion     |    string    |    服务版本，必填    |
|   frameworkVersion     |    string    |    框架版本，dolphin框架必填    |
|   tags     |    map < string,string >    |    标签列表，每个标签的名称和值都是string类型    |
|   ports     |    map < string,int >    |    除主端口之外的其他端口   |

详细说明：

1. serviceType: 容许为空，默认值是 "dolphin"，表示当前服务是 dolphin 框架的。因此在dolphin框架中这个值可以直接设为空，不用设置。主要是留给以后的，未来容许其他非dolphin框架时这个字段必须填写，具体值待定。

#### 服务实例的状态

#### 服务实例的配置
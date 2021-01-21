# Zookeeper

zookeeper=文件系统+监听通知机制

## 概念

### Znode类型

- **PERSISTENT-持久化目录节点**

  客户端与zookeeper断开连接后，该节点依旧存在

- **PERSISTENT_SEQUENTIAL-持久化顺序编号目录节点**

  客户端与zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号

- **EPHEMERAL-临时目录节点**

  客户端与zookeeper断开连接后，该节点被删除

- **EPHEMERAL_SEQUENTIAL-临时顺序编号目录节点**

  客户端与zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号

### 观察与通知

客户端向zookeeper请求，在特定的znode设置观察点（watch）。当该znode发生变化时，会触发zookeeper的通知，客户端收到通知后进行业务处理。观察点触发后立即失效。所以一旦观察点触发，需要再次设置新的观察点。

## 客户端

### Curator


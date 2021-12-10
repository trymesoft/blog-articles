Posted on 2019-02-17
> 一致性哈希算法是分布式系统常用的算法，分布式系统每个节点都有可能失效，并且新的节点很可能动态的增加进来，所以，当节点发生变化时，上面的数据要保持尽可能小的振荡，一致性哈希算法容错性及可扩展性很高。详细信息可参考这篇文章：
>
> [post cid="105" /]

### 具体实现

> 项目中使用了客户端与服务端的长连接，考虑到服务的可用性，必然要使用负载均衡，但是最常见的 Nginx 是 HTTP 的负载均衡与反向代理服务器，长连接的负载均衡还需要其他的解决方案。

最终选用了 ZooKeeper 及一致性哈希算法分配节点，具体实现如下：

1. 项目启动时在 ZooKeeper 集群中创建一个节点并同步初始化哈希环（保证哈希环节点和服务器节点同步）；
2. 客户端连接时，使用同一种 Hash 函数计算出该客户端对应的 Hash 值来分配节点，返回该客户端连接的具体节点信息并保存到 Redis 中，防止重复分配，定时过期重新分配；
3. 当哈希环中的某个节点宕机时，根据配置好的 Watcher，进行哈希环节点更新操作；
4. 当新增节点时，重复步骤 1。

> `Watcher`：ZooKeeper 支持一种 Watch 操作，Client 可以在某个 ZNode 上设置一个 Watcher，来 Watch 该 ZNode 上的变化。如果该 ZNode 上有相应的变化，就会触发这个 Watcher，把相应的事件通知给设置 Watcher 的 Client。需要注意的是，ZooKeeper 中的 Watcher 是一次性的，即触发一次就会被取消，如果想继续 Watch 的话，需要客户端重新设置 Watcher。

### 代码分析

ZooKeeper 初始化及创建节点伪代码如下：

```java
private void init() {
    try {
        zk = new ZooKeeper(conf.getConnectionStr(), conf.getSessionTimeout(), new ZookeeperWatcher());

        if (null == zk.exists(conf.getZkNode(), true)) {
            zk.create(conf.getZkNode(), "".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        }
    } catch (Exception e) {
        e.printStackTrace();
    } 
}

public void createNode(String node, String parentNode) throws Exception {
    logger.info("zookeeper创建节点 => {}", node);
    zk.create(parentNode + "/" + node, "".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
}
```

自定义的 ZookeeperWatcher，：

```java
private class ZookeeperWatcher implements Watcher {

    @Override
    public void process(WatchedEvent event) {
        logger.info("注册中心收到事件 => {}", event);

        if (KeeperState.SyncConnected == event.getState() && null == event.getPath()) {
            logger.info("已连接zookeeper服务器");
            cdl.countDown();
        } else if (EventType.NodeChildrenChanged == event.getType()) {
            logger.info("节点 {} {}", event.getPath(), "的子节点发生变更...");
            if (!conf.getZkNode().equals(event.getPath())) {
                return;
            }
            List<String> children = regCenter.getChild();
            logger.info("更新哈希环列表 => {}", children);
            ConsistencyHash.init(children);
        }
    }
}
```

一致性哈希算法的初始化：

```java
    /**
     * 真实服务器信息+虚拟节点服务器信息
     */
    private static TreeMap<Long, String> nodes = new TreeMap<>();

    /**
     * 真实服务器节点信息
     */
    private static List<String> shards = new ArrayList<>();

    /**
     * 设置虚拟节点数目(每个真实负载节点同时存在的虚拟节点)
     */
    private static int VIRTUAL_NUM = 2;

    /**
     * 初始化一致性 hash 环
     *
     * @param shards 真实服务器节点信息列表
     */
    public static void init(List<String> shards) {
        TreeMap<Long, String> nodes = new TreeMap<Long, String>();
        for (int i = 0; i < shards.size(); i++) {
            String shardInfo = shards.get(i);
            for (int j = 0; j < VIRTUAL_NUM; j++) {
                nodes.put(hash(computeMd5("SHARD-" + i + "-NODE-" + j), j),
                        shardInfo);
            }
        }
        ConsistencyHash.nodes = nodes;
        ConsistencyHash.shards = shards;
    }
```

根据 key 的 hash 值分配节点：

```java
    /**
     * 根据key的hash值取得服务器节点信息
     *
     * @param hash
     * @return
     */
    public static String getShardInfo(long hash) {
        Long key = hash;
        SortedMap<Long, String> tailMap = nodes.tailMap(key);
        if (tailMap.isEmpty()) {
            key = nodes.firstKey();
        } else {
            key = tailMap.firstKey();
        }
        return nodes.get(key);
    }
```

实测，分配 10 万个 key，大致可以均匀分布在 3 个节点上。

### 总结

ZooKeeper 集群至少 3 台，才能保证不低于一半节点工作，ZooKeeper 的主要作用是在动态添加或者节点宕机时无需手动或代码侵入的同步哈希环，通过其内部的 Watcher 机制即可方便的实现。

一致性哈希算法是分配节点信息的主算法，在分布式环境节点变化时防止某个节点负载突然变大并且尽可能的均匀分配。


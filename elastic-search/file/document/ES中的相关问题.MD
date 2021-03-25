# Elastic Search([详见官方文档](https://elastic.co))

> ES启动以集群为单位一个节点也称为集群。ES为去中心化集群，有一个所谓的主节点用于管理元数据的(其他节点IP、端口、名称...)。

> ES通过创建索引存数据。有点类似MongoDB中的Collection。数据库中的表格。

> ES有两种分片(primary shard(主分片) 、replicas(副本分片))。一般的索引创建后需要分片(默认5个主)至少创建一个主分片，副本创建建议0-n个(默认1个副)
> 主分片权限：R W。副分片权限：R。主分片会有路由策略，用于存储和精确搜索。

> 在ES5.x版本以下，一个索引有多个TYPE；ES6.x，一个索引只能有一个TYPE；ES7.x，取消TYPE定义。

> 文档是ES中最小的数据单元(类似一个JSON)。有多个属性。

     使用案例
        维基百科、The Guardian、Stack Overflow（程序异常讨论网站）、Github

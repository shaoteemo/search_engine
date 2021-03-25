#How to use ES

1.Es需使用小写字母，不能以-、_、+开头。

  在当前版本中创建索引默认创建一个主分片(primary shard)(ver.<7.x为5个)，每个主分片对应1个副分片.
  
  当磁盘空间不足15%时，不分配replica shard。当不足5%，不分配任何primary shard。
  
2.相同的分片不可以在同一个节点上，否则视为多余的，会报集群警告。因此需要计算分片与集群数量的关系。
##注意下面的语句全部在Kibana的DevTools中进行。使用RESTFulApi

####创建一个索引(有就修改，没有就添加)
`PUT [索引名称]`

####查看分片信息
`GET _cat/shards`

####修改主分片创建数及副分片创建数
`PUT [索引名称]
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 0
  }
}`

####修改索引
  注意:只能修改副本分片数量
`PUT [索引名称]/_settings
{
  "number_of_replicas": 1
}`

####删除索引
`DELETE [索引名称],[索引名称]`

> 注：_cat查看各种信息参数

####查看服务器状态
`GET _cat/health?v`参数v代表显示变量名
####状态中的green表示一切正常。yellow表示有副本分片未激活。red表示错误，主分片异常。
`测试yellow情况发生测试
PUT default_index/_settings
{
  "number_of_replicas": 2
}`

####查询索引
`GET _cat/indices?v`

####查询分片信息
`GET _cat/shards?v`

####查询节点信息
`GET _cat/nodes?v`


##文档操作
> 当前版本创建文档不需要指定类型(>6.x <7.x需要一个类型)

####新增文档(7.x)
  注：其他版本详见文档
  
  该版本类型type类型有且仅有一个类型：_doc

  1./{索引名称（下称：index）}/_doc/{id}
  
    PUT shaoteemo/_doc/2
    {
      "name":"shaoteemo2",
      "age":24,
      "gender":"male"
    }
    
  2./{index}/_doc
  
    POST shaoteemo/_doc
    {
      "name":"shaoteemo3",
      "age":24,
      "gender":"male"
    }
    
  3./{index}/_create/{id}  --强制新增。主键存在报错，不存在新增
  
    POST[PUT] shaoteemo/_create/6
     {
       "name":"shaoteemo5",
       "age":24,
       "gender":"male"
     }
    
  4.未创建的索引的新增,可用。
  
  注：ID不指配时ES会创建GUID(UUID算法强化。用于分布式不出现重复的id(当前索引)。)。如果使用相同主键则更新(3点除外)
  POST可以不用传递id自动生成。
  
####查询文档(7.x)

元数据以：_XXX命名

1.根据主键查询文档：/{index}/_doc[其他版本对应自定义的类型]/{id}

`GET shaoteemo/_doc[其他版本对应自定义的类型]/1`

2.多条件查询。可以在不同的集合中查询不同的文档数据

    GET _mget
    {
      "docs":[
        {
          "[ES维护元数据的键名]":"值",
          "[ES维护元数据的键名]":值
        },
        {
          "_index":"shaoteemo",
          "_id":2
        }
      ]
    }

####修改文档(7.x)

注：ES更新文档过程。标记修改删除文档>版本号在原有基础上+1。该操作会提升效率。然而标记删除并不会立即删除，
版本号(_version)更新从0开始的时候，ES会在空闲的时候然后从磁盘删除。ES是多线程即使有较慢的操作也可以忽略不计。

1.数据修改之全量替换,即用新数据替换原有的数据(较快)。/{index}/_doc/{id}

    PUT shaoteemo/_doc/1
    {
      "name":"修改过后的ShaoTeemo",
      "age": 200
    }

    POST shaoteemo/_doc/1
    {
      "name":"修改过后的ShaoTeemo",
      "age": 200,
      "gender":"男"
    }

2.部分更新,更新已存在的字段，添加不存在的字段：/{index}/_update/{id}

注：部分更新文档过程。标记删除>查询原始数据并比对>整合数据>替换(较慢)

    POST shaoteemo/1/_update
    {
      "doc":{
        "name": "部分替换",
        "admin": 1
      }
    }
    
####删除文档数据(7.x)

注：删除文档也是打标记，空闲时才物理删除。

    DELETE [索引名称]/[类型]/[ID]
    
#####批量操作(增、删、改)*
注：格式要求，一个请求必须是一行，以提升效率(服务器不用拼接字符串)。底层使用字符流读取，一次读取一行。

不会因为一个请求执行错误导致下面的语句执行失败。

    POST _bulk
    #强制插入
    {"create":{"_index":"shaoteemo","_id":"10086"}}
    #插入的内容
    {"name":"强制新增，批量的"}
    #全量替换
    {"index":{"_index":"shaoteemo","_id":"10087"}}
    #替换内容
    {"name":"全量替换没有就添加，批量的"}
    #全量替换
    {"index":{"_index":"shaoteemo","_id":"6"}}
    {"name":"覆盖(全量替换)，批量的"}
    #更新
    {"update":{"_index":"shaoteemo","_id":"2"}}
    {"doc":{"name":"更新不替换"}}
    #删除
    {"delete":{"_index":"shaoteemo","_id":"8SCgkHQBBif4Gzl1yEKz"}}
    
注意事项：批量操作_bulk请求体有性能瓶颈，受CPU（高速缓存）、内存、网络影响。推荐大小5-15MB。

####分词器及标准化处理(7.x)

#####常用分词器全为印欧语系类分词器
<u>standard</u> analyzer：默认的分词器，当text为文本时，使用默认的分词器。保留部分的符号。保留数字。

<u>simple</u> analyzer：符号数字全部删除，可能破坏寓意

<u>whitespace</u> analyzer：空白符分割，可能会破坏语义

######语言分词器(language analyzer)
<u>english</u> analyzer：忽略介词等，忽略复数等词性等

<u>chinese</u> analyzer：内置中文分词器，一字一词。

    GET _analyze
    {
      "text": "你好你在干嘛？[句式]"
      , "analyzer": "chinese[分词器]"
    }
    
#####中文分词器使用
######安装
1.需要下载后

[Github地址](https://github.com/medcl/elasticsearch-analysis-ik)
[Gitee地址](https://gitee.com/mirrors/elasticsearch-analysis-ik?_from=gitee_search)

2.修改pom.xml中的`<elasticsearch.version>`版本为自己的安装ES的版本。编译。

3.用Maven命令`mvn clean package`打包安装使用

4.授权文件所有人到非root用户(与安装ES授予的用户一致)

5.打包后把.zip包解压放到es根目录/plugins/目录下。安装完成，重启服务器即可。

######IK分词器使用及介绍
<u>ik_smart</u> analyzer：简短分词。

<u>ik_max_word</u> analyzer：分词更细。

    GET _analyze
    {
      "text": "50年前由长沙土夫子（盗墓贼）出土的战国帛书，记载了一个奇特战国古墓的位置，50年后，其中一个土夫子的孙子在他的笔记中发现这个秘密，纠集了一批经验丰富的盗墓贼前去寻宝，谁也没有想到。"
      , "analyzer": "ik_max_word"
    }

######IK分词器配置介绍

配置文件：IKAnalyzer.cfg.xml中可以配置自定义的词典。配置时只放入文件名。

词典的加载：配置文件中以"extra_"开头的不是必须加载词典需要配置后加载。其他的是必须加载的词典(自身没                                  有新增字典，没有修改过文件名)

.dic词库应该是：UTF-8的字符集

    main.dic--主词库
    quantifier.dic--量词
    ……
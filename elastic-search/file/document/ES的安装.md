# Elastic Search Install For Linux
## 当前环境版本

  Linux:CnetOS 8内核版本有要求详见官方文档。
  查看当前其他用户方式:`uname -a`
  ES版本:7.9(6.x为稳定版本)

## Linux中的环境配置

  1.修改`/etc/security/limits.conf` 文件(ES最少需要创建4096个线程权限，65536个资源，创建的文件数65535)
  
     -|---------------------------------------------------|-
      |*               soft    nofile          65536      |
      |*               hard    nofile          65536      |
      |*               soft    nproc           4096       |
      |root            soft    nproc           unlimited  |
     -|---------------------------------------------------|-
     
  2.开启Linux虚拟内存
      修改`/etc/sysctl.d/99-sysctl.conf`(控制配置文件)分配虚拟内存。注：Linux默认不开启
                  `vm.max_map_count=6553600`
      `sysctl -p`刷新配置
        
## 安装

  ES不能在root用户下使用，需要转移用户信息。
    
  使用：`ls /home/`查看用户组信息
    
  使用：`chown -R [用户名].[用户名] /usr/MyApp/elasticsearch-7.9.0` 将权限交给非root用户
  注意不要使用root用户操作修改好的权限移交
    
  使用`su [用户名]` 完成用户切换
  
  #### 一定要在非root下运行
    
  修改conf目录下的elasticsearch.yml
      添加--Node--：`node.name: node-1`(必须指配)
      添加--Network-- :`network.host: 0.0.0.0`
      修改任意客户端访问否则只能本地。
      添加--Discovery-- :`cluster.initial_master_nodes: ["node-1"]`(名称匹配)
        
  启动bin目录中的elasticsearch
    
  浏览器访问[查看效果](http://192.168.84.25:9300)或Linux中：`curl http://[IP]:9200`查看效果 
  
----------------------------------------------------------------------------------------------------------------

## 集群版（待测试）

  伪集群：复制一份直接启动，当前版本注意修改Discovery
  
  正常集群：
    前面正常安装单机。
>    [当前版本]修改--Discovery--配置集群`discovery.seed_hosts: ["ip1","ip2"]` 中间只写IP
>    [当前版本]修改--Discovery--配置集群`cluster.initial_master_nodes: ["node-1","node-2"]`
>    [<7.x]修改--Discovery--配置集群`discovery.zen.ping.unicast.hosts:["ip1","ip2"]`
>    [<7.x]修改--Discovery--配置集群`discovery.zen.minimum_master_nodes:最小集群数`最小集群数计算公式：集群总数/2+1
    
  测试
    访问集群任意节点其中：cluster_name与cluster_uuid应该是一致的。至此集群搭建成功
    
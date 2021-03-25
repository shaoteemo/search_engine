# Kibana Install For Linux

  系统：CentOS8
  版本：7.9.1

##安装
  1.解压即可用 修改conf中的配置文件kibana.yml文件配置`server.host: "0.0.0.0"`为任意访问(测试用)
  
  ####注意kibana当前版本不可以使用root用户运行(<7.x除外)
  为文件夹服务非root账户权限
  
  2.测试[访问浏览器](http://ip:5601)即可
  
  3.配置`i18n.locale: "zh-CN"`可以切换中文界面(翻译不完整。不推荐！)
  
  4.启动UUID错误。读写权限不够使用chmod 770授权
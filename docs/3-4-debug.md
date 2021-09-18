### 管理端问题

> server部署完成后，web页面始终无法打开

解决办法：

1. 确认浏览器访问地址是 https://[server]:4433/web/login，注意不可缺少“/web/login”这个路径

2. 确认server进程的运行情况和4433端口开放情况，如果不正常需要重启server进程，并记录错误日志

   ```shell
   # 检查./server的进程是否运行正常
   ps ax | grep ./server | grep -v grep
   
   # 检查tcp/4433端口是否正常开放
   ss -ntpl
   
   蜜罐服务名称.log
   ```

3. 检查server主机是否开启了防火墙，导致目前无法访问

4. 如果以上都没有问题，请将server和client日志提供给我们

   ```shell
   # 节点端日志在安装目录的logs文件夹内，文件名为client.log
   # 控制端日志在安装目录的logs文件夹内，文件名为server-*.log
   ```



### 节点问题

> 在机器上执行了节点脚本，但是「节点管理」处没有信息

检查管理端防火墙以及ACL策略是否放行了节点对server 4434端口的访问

```shell
# 可以从节点主机发起wget测试
wget [服务器地址]:4434
```

> 节点状态为红色离线

![](http://img.threatbook.cn/hfish/20210812135754.jpg)

解决办法：

1. 检查节点到管理端的网络连通情况，并等待……

   ```shell
   节点每90秒连接server的TCP/4433端口一次，180秒内连接不上，即显示离线。
   在刚刚完成部署或网络不稳定的时候会出现这种情况。
   通常情况，等待2~3分钟，如果节点恢复绿色在线，那蜜罐服务也会从绿色启用，变成绿色在线。
   ```


2. 如果确认网络访问正常，节点在server上始终离线，需要检查节点上的进程运行情况。如果进程运行异常，需要杀死全部关联进程后，重启进程，并记录错误日志。

   ```shell
   # 检查./client的进程是否运行正常
   ps ax | grep -E 'services|./client' | grep -v grep
   
   # 检查./service的进程是否运行正常	
   ps ax | grep ./server | grep -v grep
   ```

   

### 蜜罐服务问题



> 节点在线，部分蜜罐服务在线，部分蜜罐服务离线

解决办法：

1. 确认蜜罐服务进程是否还在运行？

   ```shell
   # 检查service的进程是否运行正常，如果进程退出，建议查看service的日志
   ps ax | grep service | grep -v grep
   ```

2. 确认是否端口冲突？

   ```shell
   这个问题常见默认22端口的SSH服务，刚启动client的时候，服务在线，过了一会儿后服务离线。
   用ss -ntpl检查该蜜罐服务的端口是否被占用？如果被占用，建议修改该业务的默认端口。
   
   在Windows操作系统上，如果用户启用了tcp端口监听，大概率会发现tcp 135、139、445、3389端口冲突，
   这是用于Windows默认占用了这些端口，不太建议在Windows上使用tcp 135、139、445、3389端口的蜜罐。
   ```

> 变更服务模板后，蜜罐新服务访问不到	

```
在HFish当前的产品结构中，管理端永远不会主动连接节点进行节点配置的变更。
而是在管理端上生成一个配置，等待节点来拉取。
节点每90秒尝试连接管理端一次，获取到变更数据后，还需要从管理端上拉取新的服务，解压服务包并运行。
运行服务的结果，会在下一个90秒回连时，上报到管理端。
这个流程最慢的话，可能会有一个3分钟左右延时。
所以刚刚变更蜜罐服务后，请大家稍微等等。
```

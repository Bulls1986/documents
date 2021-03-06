# [分布式存储方案Minio]



# 一、简介

Minio 是一个基于 golang 语言开发的 AWS S3 存储协议的开源实现，并附带 web ui 界面，可以通过 Minio 搭建私人的兼容 AWS S3 协议的存储服务器。Minio可以做为云存储的解决方案用来保存海量的图片，视频，文档。由于采用Golang实现，服务端可以工作在Windows,Linux, OS X和FreeBSD上。配置简单，基本是复制可执行程序，单行命令可以运行起来。

# 二、Minio 服务器搭建

## 1.直接运行编译后的文件启动

Minio 基于 golang 开发，所以编译后只有一个可执行文件，启动一个 Minio 服务器极其简单，只需要使用 `server` 参数，并附带一个或多个存储目录即可。

col 1                                                                                                                                                                                                                                       
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
`# 下载 Minio`

`wget https:``//dl.minio.io/server/minio/release/linux-amd64/minio`

`# 赋予可执行权限`

`chmod +x minio`

`# 创建一个目录用于存放 minio 文件`

`mkdir -p /home/minio/data/`

`# 以后台方式启动一个 minio 服务器`

`nohup ./minio server /home/minio/data/ &`

minio 默认监听所有网卡的 9000 端口，此时直接访问 `[http://ip:9000](http://ip:9000/)` 即可查看 web ui 界面，如下所示

同时在启动 minio 后默认会输出当前 minio 服务器的相关登录参数，如 access_key 等，nohup 启动则默认重定向到了 nohup.out 文件中，如下所示

## 2.通过docker启动

创建数据存放的文件夹以及配置文件存放文职

col 1                                                       
------------------------------------------------------------
`mkdir -p /home/minio/data/`

`mkdir -p /home/minio/config/`

创建配置文件

col 1                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
`{`

`        ``"version"``: ``"18"``,`

`        ``"credential"``: {`

`                ``"accessKey"``: ``"admin"``,`

`                ``"secretKey"``: ``"admin123"`

`        ``},`

`        ``"region"``: ``""``,`

`        ``"browser"``: ``"on"``,`

`        ``"logger"``: {`

`                ``"console"``: {`

`                        ``"enable"``: ``true`

`                ``},`

`                ``"file"``: {`

`                        ``"enable"``: ``false``,`

`                        ``"filename"``: ``""`

`                ``}`

`        ``},`

`        ``"notify"``: {`

`                ``"amqp"``: {`

`                        ``"1"``: {`

`                                ``"enable"``: ``false``,`

`                                ``"url"``: ``""``,`

`                                ``"exchange"``: ``""``,`

`                                ``"routingKey"``: ``""``,`

`                                ``"exchangeType"``: ``""``,`

`                                ``"deliveryMode"``: ``0``,`

`                                ``"mandatory"``: ``false``,`

`                                ``"immediate"``: ``false``,`

`                                ``"durable"``: ``false``,`

`                                ``"internal"``: ``false``,`

`                                ``"noWait"``: ``false``,`

`                                ``"autoDeleted"``: ``false`

`                        ``}`

`                ``},`

`                ``"nats"``: {`

`                        ``"1"``: {`

`                                ``"enable"``: ``false``,`

`                                ``"address"``: ``""``,`

`                                ``"subject"``: ``""``,`

`                                ``"username"``: ``""``,`

`                                ``"password"``: ``""``,`

`                                ``"token"``: ``""``,`

`                                ``"secure"``: ``false``,`

`                                ``"pingInterval"``: ``0``,`

`                                ``"streaming"``: {`

`                                        ``"enable"``: ``false``,`

`                                        ``"clusterID"``: ``""``,`

`                                        ``"clientID"``: ``""``,`

`                                        ``"async"``: ``false``,`

`                                        ``"maxPubAcksInflight"``: ``0`

`                                ``}`

`                        ``}`

`                ``},`

`                ``"elasticsearch"``: {`

`                        ``"1"``: {`

`                                ``"enable"``: ``false``,`

`                                ``"format"``: ``""``,`

`                                ``"url"``: ``""``,`

`                                ``"index"``: ``""`

`                        ``}`

`                ``},`

`                ``"redis"``: {`

`                        ``"1"``: {`

`                                ``"enable"``: ``false``,`

`                                ``"format"``: ``""``,`

`                                ``"address"``: ``""``,`

`                                ``"password"``: ``""``,`

`                                ``"key"``: ``""`

`                        ``}`

`                ``},`

`                ``"postgresql"``: {`

`                        ``"1"``: {`

`                                ``"enable"``: ``false``,`

`                                ``"format"``: ``""``,`

`                                ``"connectionString"``: ``""``,`

`                                ``"table"``: ``""``,`

`                                ``"host"``: ``""``,`

`                                ``"port"``: ``""``,`

`                                ``"user"``: ``""``,`

`                                ``"password"``: ``""``,`

`                                ``"database"``: ``""`

`                        ``}`

`                ``},`

`                ``"kafka"``: {`

`                        ``"1"``: {`

`                                ``"enable"``: ``false``,`

`                                ``"brokers"``: ``null``,`

`                                ``"topic"``: ``""`

`                        ``}`

`                ``},`

`                ``"webhook"``: {`

`                        ``"1"``: {`

`                                ``"enable"``: ``false``,`

`                                ``"endpoint"``: ``""`

`                        ``}`

`                ``},`

`                ``"mysql"``: {`

`                        ``"1"``: {`

`                                ``"enable"``: ``false``,`

`                                ``"format"``: ``""``,`

`                                ``"dsnString"``: ``""``,`

`                                ``"table"``: ``""``,`

`                                ``"host"``: ``""``,`

`                                ``"port"``: ``""``,`

`                                ``"user"``: ``""``,`

`                                ``"password"``: ``""``,`

`                                ``"database"``: ``""`

`                        ``}`

`                ``}`

`        ``}`

`}`

执行命令启动minio

col 1                                                                                                                                                                                
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
`docker run --rm --net=host --name minio --privileged=``true` `-v /home/minio/data:/export:rw -v /home/minio/config:/root/.minio/:rw -p ``9000``:``9000` `minio/minio server /export`

将docker启动的minio做成服务，并开机启动

在/usr/lib/systemd/system/ 下面创建minio.service

col 1                                                                                                                                         
----------------------------------------------------------------------------------------------------------------------------------------------
`cd /usr/lib/systemd/system/`

`touch minio.service `

`#将minio.serviceA 文件变成可执行权限`

`chmod +x minio.service`

`#设置开机启动`

`chkconfig minio on`

minio.service文件内容

col 1                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
`[Unit]`

`Description=minio`

`After=docker.service`

`Requires=docker.service`

`Requires=dnsmasq.service`

`[Service]`

`Restart=always`

`RestartSec=``20`

`TimeoutStartSec=20m`

`ExecStartPre=-/usr/bin/docker pull minio/minio:latest`

`ExecStart=/usr/bin/docker run \`

`    ``--rm \`

`    ``--net=host \`

`    ``--name minio \`

`    ``--privileged=``true` `\`

`    ``-v /home/minio/data:/export:rw  \`

`    ``-v /home/minio/config:/root/.minio/:rw  \`

`    ``-p ``9000``:``9000` `\`

`    ``minio/minio server /export`

`ExecStop=/bin/bash -c " \`

`    ``/usr/bin/docker kill minio/minio && \`

`    ``/usr/bin/docker rm -f minio/minio "`

`[Install]`

`WantedBy=multi-user.target`

启动minio

col 1                                                                  
-----------------------------------------------------------------------
`#查看minio状态`

`service minio status`

`#启动minio`

`service minio start`

# 三、搭建Minio高可用环境

注意：Minio的高可用在存储数据的时候，采用的是 Erasure Code 技术，需要至少4个最多16个驱动

需要在2台机器上面搭建Minio的高可用环境

node186:192.168.2.186

node151:192.168.2.151

1.直接运行在linux上面

分别在node186和node151上面下载Minio，按照2.1（直接运行编译后的文件启动）所示，然后在node186和node151上面分别执行


col 1                                                                                                                                                                                                                                                                                                                                                                                                        
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
`./minio server http:``//192.168.2.186/minio-export1 \`

`http:``//192.168.2.186/minio-export2 \`

`              ``http:``//192.168.2.186/minio-export3 \`

`http:``//192.168.2.186/minio-export4 \`

`              ``http:``//192.168.2.151/minio-export1 \`

`http:``//192.168.2.151/minio-export2 \`

`              ``http:``//192.168.2.151/minio-export3 \`

`http:``//192.168.2.151/minio-export4 \`

注意：2台机器的时间一定要同步，可以安装ntp让时间同步，ntp安装好后，通过命令： service ntpd status 查看ntp是否启动。

2.在docker里面运行minio的高可用

分别在node186和node151上面获取minio的镜像


col 1                    
-------------------------
`docker pull minio/minio`

然后分别启动minio


col 1                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
`docker run -p ``9000``:``9000` `--rm --name minio \`

`  ``--net=host \`

`  ``-e ``"MINIO_ACCESS_KEY=admin"` `\`

`  ``-e ``"MINIO_SECRET_KEY=admin123"` `\`

`  ``-v /home/minio/minio-export1:/export1 \`

`  ``-v /home/minio/minio-export2:/export2 \`

`  ``-v /home/minio/minio-export3:/export3 \`

`  ``-v /home/minio/minio-export4:/export4 \`

`  ``-v /home/minio/config/minio:/root/.minio \`

`  ``minio/minio server  \`

`http:``//192.168.2.186/export1 http://192.168.2.186/export2 http://192.168.2.186/export3 http://192.168.2.186/export4 \`

`http:``//192.168.2.151/export1 http://192.168.2.151/export2 http://192.168.2.151/export3 http://192.168.2.151/export4`

可以通过页面访问任何一台机器的后端管理界面，然后创建一个**bucket** ，上传一个文件进行测试，发现另外一个机器上面同样存在数据。

注意：

1.在minio的集群中，如果宕掉的节点是集群节点的一半以上，则Minio不能正常工作，需要存活的节点数大于N/2 + 1 才能正常工作，可以用三台机器进行测试，每台机器启动2个节点，然后停掉一台机器，由于存活的节点数为4，大于6/2=3，则能正常工作。

2.当节点数一旦确定，且在工作，那么不能进行动态增加节点。例如：开始是3台机器，每台机器2个节点，现在需要变成2台机器，每个机器4个节点，那么启动会报错，


col 1                                                                                                                                                                                                 
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
`time=``"2017-06-28T07:34:02Z"` `level=fatal msg=``"Initializing object layer failed"` `cause=``"Number of disks 8 did not match the backend format 6"` `source=``"[server-main.go:278:serverMain()]"`

疑似在每个数据存储的地方有对应的节点数配置，在每个数据存储的文件夹下面都有对应的一个json文件，里面有对应节点的ID。

format.json文件内容如下：


col 1                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
`[root``@pi186` `minio]# less minio-export1/.minio.sys/format.json `

`{``"version"``:``"1"``,``"format"``:``"xl"``,``"xl"``:{``"version"``:``"1"``,``"disk"``:``"1140c388-061c-496b-842e-4768e552a641"``,``"jbod"``:[``"8a3a3dad-19bb-4c30-ab4c-a45d0036b2af"``,``"62b347c1-9aa6-4d21-968d-6a852e3d12d9"``,``"7f4ff719-571a-41f7-8a3d-4f3960b37d21"``,``"5f3d14b2-294c-4723-84bf-e90464dece18"``,``"1140c388-061c-496b-842e-4768e552a641"``,``"67d3a7ef-93fd-4f92-b89a-76b2d6386cb3"``]}}`

# 四、高可用测试结果

测试前提条件

三台机器（M1，M2，M3），每台机器2个节点，分别为（M1N1，M1N2，M2N1，M2N2，M3N1，M3N2）

依次分别将三台机器的minio启动，然后做如下测试。


col 1                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
`1``.将M1N1的文件夹更改名字后能正常工作（例如：mv minio-export1 minio-export1-bak，然后添加一个test3的bucket，发现在minio-export1-bak中有test3的文件夹，）。`

`2``.将M1N1的文件夹copy一份后能正常工作，生效的是原文件夹，copy后的文件夹不生效（例如：cp -r minio-export1 minio-export1-copy，然后添加一个test4的bucket，发现在minio-export1中有test4的文件夹，而minio-export1-copy中没有）`

`3``.将M1N1的文件夹rm后偶尔不能正常工作（创建test5的bucket不成功），不成功的时候可能是访问的刚好是当前这个节点。`

`4``.重启M1的minio后单个节点能正常工作（重启得到的信息是``5``个online，``1``个offline），能读能写`

`5``.删除M1的``2``个节点的文件夹，还是能正常工作，能读能写，应该是路由到其他节点`

`6``.重启M1的minio（重启得到的信息是``4``个online，``2``个offline），能正常工作，但是没有做数据同步，M1上面的``2``个节点没有数据。`

`7``.从新启动三台机器，在M1上面创建test1，停掉M2，在M3上面创建test2,test3，发现M1和M3的每个节点都有test1、test2、test3，M2只有test1，重启M2，发现M2的每个节点都有test1、test2、test3。`

`换句话说，当某个机器宕掉后，再重启，会进行数据同步。`

总结：如果所在的机器宕掉后，文件已经造成破坏，想要恢复则必须要在其对应的节点文件夹下面加入 .minio.sys文件夹，里面包含format.json文件，可以通过其他机器反向找到当前机器的id。手动配置好 .minio.sys后，启动宕掉的机器，数据就能从其他机器上面同步了。

# 五、其他

通过nginx做负载均衡（略）

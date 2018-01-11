# redis cluster 安装


### 1、预装ruby(root用户安装)
下载地址：http://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.2.tar.gz
安装ruby2.2  

    tar xzvf  ruby-2.2.2.tar.gz
    cd /root/ruby-2.2.2
    ./configure -prefix=/usr/local/ruby  
    make  
    make install  
    cp ruby /usr/local/bin 

### 2、安装rubygem
下载地址：http://production.cf.rubygems.org/rubygems/rubygems-2.4.6.tgz

    tar xzvf  rubygems-2.4.6.tgz
    cd /root/rubygems-2.4.6 
    ruby setup.rb  
    cp bin/gem /usr/local/bin 

### 3、安装gem-redis插件（要安装zlib-devel包）
安装ruby自身提供的zlib包 

    cd ruby-2.2.2/ext/zlib
    ruby ./extconf.rb
    make
    make install
    gem install redis --version 3.0.0  （不用执行）
#由于源的原因，可能下载失败，就手动下载下来安装  
#直接下载：https://rubygems.org/gems/redis/versions/3.2.1  ; 点击donwload后去掉链接的s
#然后安装

    gem install -l /root/redis-3.2.1.gem  

### 4、redis软件的安装
a、下载

    http://download.redis.io/releases/redis-3.0.2.tar.gz
b、解压

    tar xzvf  redis-3.0.2.tar.gz
c、编译

    cd /root/redis-3.0.2
    make
d、建立目录

    mkdir -p /opt/redis30/bin /opt/redis30/var /opt/redis30/etc /opt/redis30/data  /opt/redis30/log
e、复制文件

    cd /root/redis-3.0.2/src
    cp  redis-benchmark  redis-check-aof redis-check-dump  redis-cli  redis-sentinel  redis-server  redis-trib.rb /opt/redis30/bin/
f、安装集群软件
创建集群目录

    cd /opt/
    mkdir rediscluster1 rediscluster2
复制文件

    cp -fr redis30  rediscluster1/
    cp -fr redis30  rediscluster2/

g、修改配置文件/opt/cluster/7000-5/redis30/etc/redis.conf（#不同的实例用7000至5）

    daemonize yes
    pidfile /opt/rediscluster1/redis30/log/r7000.pid
    port 7000
    
    loglevel warning
    
    logfile "/opt/rediscluster1/redis30/log/r7000.log"
    
    #save 900 1
    #save 300 10
    #save 60 10000
    
    dir /opt/rediscluster1/redis30/data/
    
    maxmemory 10000m
    
    cluster-enabled yes
    cluster-config-file nodes-7000.conf
    cluster-node-timeout 15000


h、启动redis

    （
    /opt/rediscluster1/redis30/bin/redis-server  /opt/rediscluster1/redis30/etc/redis.conf 
    /opt/rediscluster2/redis30/bin/redis-server  /opt/rediscluster2/redis30/etc/redis.conf 
    
    ）


i、创建redis集群

    cd  /opt/rediscluster1/redis30/bin
      ./redis-trib.rb create --replicas 1  10.173.212.106:7000 10.173.212.38:7000   10.173.212.67:7000  10.173.212.94:7000  10.47.185.210:7000  10.47.185.221:7000  10.173.212.106:7001 10.173.212.38:7001   10.173.212.67:7001  10.173.212.94:7001  10.47.185.210:7001  10.47.185.221:7001
    
注：--replicas 1表示每个节点有一个slave
执行命令后会提示你是否接受提示的配置信息，默认的是前3台作为master机器，后3台作为slave机器，输入yes，出现最后的信息表示集群已经创建好了。
输出如下：

    >>> Creating cluster
    Connecting to node 10.173.212.106:7000: OK
    Connecting to node 10.173.212.38:7000: OK
    Connecting to node 10.173.212.67:7000: OK
    Connecting to node 10.173.212.94:7000: OK
    Connecting to node 10.47.185.210:7000: OK
    Connecting to node 10.47.185.221:7000: OK
    Connecting to node 10.173.212.106:7001: OK
    Connecting to node 10.173.212.38:7001: OK
    Connecting to node 10.173.212.67:7001: OK
    Connecting to node 10.173.212.94:7001: OK
    Connecting to node 10.47.185.210:7001: OK
    Connecting to node 10.47.185.221:7001: OK
    >>> Performing hash slots allocation on 12 nodes...
    Using 6 masters:
    10.173.212.106:7000
    10.173.212.38:7000
    10.173.212.67:7000
    10.173.212.94:7000
    10.47.185.210:7000
    10.47.185.221:7000
    Adding replica 10.173.212.38:7001 to 10.173.212.106:7000
    Adding replica 10.173.212.106:7001 to 10.173.212.38:7000
    Adding replica 10.173.212.94:7001 to 10.173.212.67:7000
    Adding replica 10.173.212.67:7001 to 10.173.212.94:7000
    Adding replica 10.47.185.221:7001 to 10.47.185.210:7000
    Adding replica 10.47.185.210:7001 to 10.47.185.221:7000
    M: 7dc33a07ed823267a86e3f25215f51e579deb160 10.173.212.106:7000
       slots:0-2730 (2731 slots) master
    M: d00ec080ac21e04568232beac1b0154bb0e8f570 10.173.212.38:7000
       slots:2731-5460 (2730 slots) master
    M: b6fbd9e0d5f38295c1901cde4e04c3a721c4d41b 10.173.212.67:7000
       slots:5461-8191 (2731 slots) master
    M: a70283735d7b2be958e62f2d087d92a27e78d97f 10.173.212.94:7000
       slots:8192-10922 (2731 slots) master
    M: bb82a1c22cdaed3d59b9f8045de2326ed19418a0 10.47.185.210:7000
       slots:10923-13652 (2730 slots) master
    M: 78302915f66856355592ed2cae000464a49a3d86 10.47.185.221:7000
       slots:13653-16383 (2731 slots) master
    S: f9977c674446c8da13bc15b095e2fe96ae37d5ff 10.173.212.106:7001
       replicates d00ec080ac21e04568232beac1b0154bb0e8f570
    S: 0a83e26df72186ff0781a9dd651b9fba2171e933 10.173.212.38:7001
       replicates 7dc33a07ed823267a86e3f25215f51e579deb160
    S: 793337e43f9f0be21dc71178ee89caf58f8ad77a 10.173.212.67:7001
       replicates a70283735d7b2be958e62f2d087d92a27e78d97f
    S: 6a2540a47603980a39060c756d3b4dcff4921620 10.173.212.94:7001
       replicates b6fbd9e0d5f38295c1901cde4e04c3a721c4d41b
    S: 51a2de33cb697036390ac6cc24c9eb51c15687c6 10.47.185.210:7001
       replicates 78302915f66856355592ed2cae000464a49a3d86
    S: b8544903785725d4723839d8dd23a49c27253234 10.47.185.221:7001
       replicates bb82a1c22cdaed3d59b9f8045de2326ed19418a0
    Can I set the above configuration? (type 'yes' to accept): yes
    >>> Nodes configuration updated
    >>> Assign a different config epoch to each node
    >>> Sending CLUSTER MEET messages to join the cluster
    Waiting for the cluster to join........
    >>> Performing Cluster Check (using node 10.173.212.106:7000)
    M: 7dc33a07ed823267a86e3f25215f51e579deb160 10.173.212.106:7000
       slots:0-2730 (2731 slots) master
    M: d00ec080ac21e04568232beac1b0154bb0e8f570 10.173.212.38:7000
       slots:2731-5460 (2730 slots) master
    M: b6fbd9e0d5f38295c1901cde4e04c3a721c4d41b 10.173.212.67:7000
       slots:5461-8191 (2731 slots) master
    M: a70283735d7b2be958e62f2d087d92a27e78d97f 10.173.212.94:7000
       slots:8192-10922 (2731 slots) master
    M: bb82a1c22cdaed3d59b9f8045de2326ed19418a0 10.47.185.210:7000
       slots:10923-13652 (2730 slots) master
    M: 78302915f66856355592ed2cae000464a49a3d86 10.47.185.221:7000
       slots:13653-16383 (2731 slots) master
    M: f9977c674446c8da13bc15b095e2fe96ae37d5ff 10.173.212.106:7001
       slots: (0 slots) master
       replicates d00ec080ac21e04568232beac1b0154bb0e8f570
    M: 0a83e26df72186ff0781a9dd651b9fba2171e933 10.173.212.38:7001
       slots: (0 slots) master
       replicates 7dc33a07ed823267a86e3f25215f51e579deb160
    M: 793337e43f9f0be21dc71178ee89caf58f8ad77a 10.173.212.67:7001
       slots: (0 slots) master
       replicates a70283735d7b2be958e62f2d087d92a27e78d97f
    M: 6a2540a47603980a39060c756d3b4dcff4921620 10.173.212.94:7001
       slots: (0 slots) master
       replicates b6fbd9e0d5f38295c1901cde4e04c3a721c4d41b
    M: 51a2de33cb697036390ac6cc24c9eb51c15687c6 10.47.185.210:7001
       slots: (0 slots) master
       replicates 78302915f66856355592ed2cae000464a49a3d86
    M: b8544903785725d4723839d8dd23a49c27253234 10.47.185.221:7001
       slots: (0 slots) master
       replicates bb82a1c22cdaed3d59b9f8045de2326ed19418a0
    [OK] All nodes agree about slots configuration.
    >>> Check for open slots...
    >>> Check slots coverage...
    [OK] All 16384 slots covered.

k、集群的维护
redis-trib.rb的使用方法

    ./redis-trib.rb           
    Usage: redis-trib <command> <options> <arguments ...>
    
      create          host1:port1 ... hostN:portN
                      --replicas <arg>
      check           host:port
      fix             host:port
      reshard         host:port
      add-node        new_host:new_port existing_host:existing_port
                      --slave
                      --master-id <arg>
      del-node        host:port node_id
      set-timeout     host:port milliseconds
      call            host:port command arg arg .. arg
      import          host:port
                      --from <arg>
      help            (show this help)
    
    For check, fix, reshard, del-node, set-timeout you can specify the host and port of any working node in the cluster.

集群客户端的使用

    ./redis-cli -c -h 主机ip -p 端口 （-c是为了操作集群）
集群命令： 

    CLUSTER INFO 打印集群的信息  
    CLUSTER NODES 列出集群当前已知的所有节点（node），以及这些节点的相关信息。  
    节点  
    CLUSTER MEET <ip> <port> 将 ip 和 port 所指定的节点添加到集群当中，让它成为集群的一份子。  
    CLUSTER FORGET <node_id> 从集群中移除 node_id 指定的节点。  
    CLUSTER REPLICATE <node_id> 将当前节点设置为 node_id 指定的节点的从节点。  
    CLUSTER SAVECONFIG 将节点的配置文件保存到硬盘里面。  
    槽(slot)  
    CLUSTER ADDSLOTS <slot> [slot ...] 将一个或多个槽（slot）指派（assign）给当前节点。  
    CLUSTER DELSLOTS <slot> [slot ...] 移除一个或多个槽对当前节点的指派。  
    CLUSTER FLUSHSLOTS 移除指派给当前节点的所有槽，让当前节点变成一个没有指派任何槽的节点。  
    CLUSTER SETSLOT <slot> NODE <node_id> 将槽 slot 指派给 node_id 指定的节点，如果槽已经指派给另一个节点，那么先让另一个节点删除该槽>，然后再进行指派。  
    CLUSTER SETSLOT <slot> MIGRATING <node_id> 将本节点的槽 slot 迁移到 node_id 指定的节点中。  
    CLUSTER SETSLOT <slot> IMPORTING <node_id> 从 node_id 指定的节点中导入槽 slot 到本节点。  
    CLUSTER SETSLOT <slot> STABLE 取消对槽 slot 的导入（import）或者迁移（migrate）。  
    键  
    CLUSTER KEYSLOT <key> 计算键 key 应该被放置在哪个槽上。  
    CLUSTER COUNTKEYSINSLOT <slot> 返回槽 slot 目前包含的键值对数量。  
    CLUSTER GETKEYSINSLOT <slot> <count> 返回 count 个 slot 槽中的键。  



l、实例操作
添加一个新的master节点
添加一个master节点:创建一个空节点（empty node），然后将某些slot移动到这个空节点上,这个过程目前需要人工干预
a):创建目录和生成配置文件

    cd /opt/cluster/
    mkdir 7006
    cp -fr /opt/redis30 /opt/cluster/7006/
    创建/opt/cluster/7006/redis30/etc/redis.conf

b):启动节点

    /opt/cluster/7006/redis30/bin/redis-server  /opt/cluster/7006/redis30/etc/redis.conf 

c):加入空节点到集群
add-node  将一个节点添加到集群里面， 第一个是新节点ip:port, 第二个是任意一个已存在节点ip:port

    /opt/cluster/7006/redis30/bin/redis-trib.rb add-node 127.0.0.1:7006 127.0.0.1:7000 
    node:新节点没有包含任何数据， 因为它没有包含任何slot。新加入的加点是一个主节点， 当集群需要将某个从节点升级为新的主节点时， 这个新节点不会被选中

d):为新节点分配slot

    redis-trib.rb reshard 127.0.0.1:7006  
    #根据提示选择要迁移的slot数量(ps:这里选择4000)  
    How many slots do you want to move (from 1 to 16384)? 4000  
    #选择要接受这些slot的node-id  
    What is the receiving node ID? a9f87ba060dd961a2ea67dfbe99b5c019a6e86d3  
    #选择slot来源:  
    #all表示从所有的master重新分配，  
    #或者数据要提取slot的master节点id,最后用done结束  
    Please enter all the source node IDs.  
      Type 'all' to use all the nodes as source nodes for the hash slots.  
      Type 'done' once you entered all the source nodes IDs.  
    Source node #1:all  
    #打印被移动的slot后，输入yes开始移动slot以及对应的数据.  
    #Do you want to proceed with the proposed reshard plan (yes/no)? yes  
    #结束  

m、添加新的slave节点

    a):创建目录和生成配置文件
    cd /opt/cluster/
    mkdir 7007
    cp -fr /opt/redis30 /opt/cluster/7007/
    创建/opt/cluster/7007/redis30/etc/redis.conf

    b):启动节点
    /opt/cluster/7007/redis30/bin/redis-server  /opt/cluster/7007/redis30/etc/redis.conf 
    
    c):加入空节点到集群
    add-node  将一个节点添加到集群里面， 第一个是新节点ip:port, 第二个是任意一个已存在节点ip:port
    /opt/cluster/7006/redis30/bin/redis-trib.rb add-node 127.0.0.1:7007 127.0.0.1:7000 
    
    d)第四步:redis-cli连接上新节点shell,输入命令:cluster replicate 对应master的node-id
    可以查看节点文件知道master的node-id
    ./redis-cli -c -h 127.0.0.1 -p 7007 
    指定自己master
    cluster replicate 2b9ebcbd627ff0fd7a7bbcc5332fb09e72788835  
     
    note:在线添加slave 时，需要dump整个master进程，并传递到slave，再由 slave加载rdb文件到内存，rdb传输过程中Master可能无法提供服务,整个过程消耗大量io,小心操作.
    例如本次添加slave操作产生的rdb文件
     -rw-r--r-- 1 root root  34946 Apr 17 18:23 dump-6386.rdb  
    -rw-r--r-- 1 root root  34946 Apr 17 18:23 dump-7386.rdb  
     
    e):在线reshard 数据:
    对于负载/数据均匀的情况，可以在线reshard slot来解决,方法与添加新master的reshard一样，只是需要reshard的master节点是老节点.

o、删除一个slave节点
#redis-trib del-node ip:port '<node-id>'  

    redis-trib.rb del-node 10.10.34.14:7386 'c7ee2fca17cb79fe3c9822ced1d4f6c5e169e378'  

p、删除一个master节点

    a):删除master节点之前首先要使用reshard移除master的全部slot,然后再删除当前节点(目前只能把被删除
    master的slot迁移到一个节点上)
      
    #把10.10.34.14:6386当前master迁移到10.10.34.14:6380上  
    redis-trib.rb reshard 10.10.34.14:6380  
    #根据提示选择要迁移的slot数量(ps:这里选择500)  
    How many slots do you want to move (from 1 to 16384)? 500(被删除master的所有slot数量)  
    #选择要接受这些slot的node-id(10.10.34.14:6380)  
    What is the receiving node ID? c4a31c852f81686f6ed8bcd6d1b13accdc947fd2 (ps:10.10.34.14:6380的node-id)  
    Please enter all the source node IDs.  
      Type 'all' to use all the nodes as source nodes for the hash slots.  
      Type 'done' once you entered all the source nodes IDs.  
    Source node #1:f51e26b5d5ff74f85341f06f28f125b7254e61bf(被删除master的node-id)  
    Source node #2:done  
    #打印被移动的slot后，输入yes开始移动slot以及对应的数据.  
    #Do you want to proceed with the proposed reshard plan (yes/no)? yes  
     
    b):删除空master节点
      
    redis-trib.rb del-node 10.10.34.14:6386 'f51e26b5d5ff74f85341f06f28f125b7254e61bf'  
    
    注：删除master时，步骤是：slot重新迁移----删除slave-----删除master。

q、数据写入实例操作

    [webuser@localhost bin]$ ./redis-cli -c -h 127.0.0.1 -p 7000
    127.0.0.1:7000> set a 1
    -> Redirected to slot [15495] located at 127.0.0.1:7001
    OK
    127.0.0.1:7001> set b 1
    -> Redirected to slot [3300] located at 127.0.0.1:7000
    OK
    127.0.0.1:7000> set c 1
    -> Redirected to slot [7365] located at 127.0.0.1:7003
    OK
    127.0.0.1:7003> set d 1
    -> Redirected to slot [11298] located at 127.0.0.1:7001
    OK
    127.0.0.1:7001> set e 1
    OK
    127.0.0.1:7001> set f 1
    -> Redirected to slot [3168] located at 127.0.0.1:7000
    OK
    127.0.0.1:7000> get b
    "1"
    127.0.0.1:7000> get d
    -> Redirected to slot [11298] located at 127.0.0.1:7001
    "1"
    127.0.0.1:7001> get e
    "1"
    127.0.0.1:7001> get f
    -> Redirected to slot [3168] located at 127.0.0.1:7000
    "1"
    127.0.0.1:7000> 
    注：从面的操作上，我们可以到string被定向到不同的slot上，而slot在不同的redis上。数据是被hash分片了；而且大家可以看到能写入的只有3个master。

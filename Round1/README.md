# Round 1 介绍
一.环境准备
  a.Ubuntu hadoop2.6
  b.安装java jdk 
    sudo apt-get update 
    sudo apt-get install openjdk
    配置环境变量  sudo vim etc/profile 在最后添加
    export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk
    exportCLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/jre/lib/dt.jar:$JAVA_HOME/jre/lib/tools.jar            
    export PATH=$PATH:$JAVA_HOME/bin
  c.修改各机器 主机名sudo vim /etc/hostname 主机设为master 其他设为slaven
    修改hosts文件 sudo vim /etc/hosts
      IP.. master
      IP.. slave1
      IP.. slave2
  d.配置ssh免密码登录
      在master上ssh-keygen生成公钥并复制cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
      在各slave上ssh-keygen生成公钥，拷贝主机的公钥scp master:~/.ssh/authorized_keys  /home/hadoop/.ssh/
hadoop安装
  a.上传hadoop2.6到master 并解压 tar -zxvf  
  b.修改配置 进入 hadoop2.6/etc/hadoop目录
    1.配置 hadoop-env.sh文件 ：修改JAVA_HOME
        export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk
    2.配置 yarn-env.sh 文件  修改JAVA_HOME
        export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk
    3.配置slaves文件 增加slave节点 
         slave1 slave2
    4.配置core-site.xml文件  添加hadoop配置（设置端口号，之前要新建文件tmp）
      <property>
                <name>fs.defaultFS</name>
                <value>hdfs://master:9000</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>file:/home/hadoop/hadoop-2.6.5/tmp</value>
                <description>Abase for other temporary directories.</description>
        </property>
    5.配置  hdfs-site.xml 文件- 增加hdfs配置信息（namenode、datanode端口和目录位置）
         <property>
           <name>dfs.namenode.secondary.http-address</name>
           <value>master:50090</value>
          </property>

           <property>
            <name>dfs.namenode.name.dir</name>
            <value>file:/home/hadoop-2.6.0/tmp/dfs/name</value>
          </property>

          <property>
           <name>dfs.datanode.data.dir</name>
           <value>file:/home/hadoop-2.6.0/tmp/dfs/data</value>
           </property>

          <property>
           <name>dfs.replication</name>
           <value>2</value>
           </property>
    6.配置  mapred-site.xml 文件
          <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
          </property>
         
    7.配置  yarn-site.xml 
     <configuration>
      <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>master</value>
      </property>
      <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
      </property>
    </configuration>
    8.配置好后将master上得hadoop复制到各个节点上
    9.启动 在master上进进入hadoop/sbin 目录下
       ./hdfs namenode -format 
       ./start-dfs.sh
       ./start-yarn.sh
    10.查看  通过jps查看各节点启动的进程 
        master 3815 NameNode               slave    3510 NodeManager                                 
                4009 SecondaryNameNode              3390 DataNode
                4330 Jps                            3606 jps
                4143 ResourceManager
数据存储： 可使用hbase作为数据存储工具。
     hbase安装配置：
          上传hbase安装包到master解压，进入 conf目录
          a.配置hbase-env.sh 添加
               export JAVA_HOME=/home/hadoop/jdk1.6.0_45/
               export HBASE_CLASSPATH=/home/hadoop/hbase-0.98.0-hadoop1/conf
               export HBASE_OPTS="-XX:+UseConcMarkSweepGC"
               export HBASE_MANAGES_ZK=true
          b.配置hbase-site.xml 添加
                <configuration>
                <property>
                 <name>hbase.master</name>
                 <value>master:6000</value>
                </property>
                <property>
                 <name>hbase.rootdir</name>
                 <value>hdfs://master:9000/hbase</value>
                </property>
                <property>
                 <name>hbase.cluster.distributed</name>
                 <value>true</value>
                </property>
                <property>
                 <name>hbase.zookeeper.quorum</name>
                 <value>master</value>
                </property>
                <property>
                 <name>hbase.zookeeper.property.dataDir</name>
                 <value>/home/hadoop/tmp/zookeeper</value>
                </property>
                </configuration>
           c.配置regionservers  添加 slave1 slave2
     hbase数据写入：
            对日志文件 每行按空格进行 split切分，切出来的第一个数据为IP  剩下的为value 
         创建hbase表，以每天的日期为rowkey ，CF：IP存储切分出来的IP ，CF：values 存储切分出来的value（建表的时候指定maxversions）
            
数据处理
    a.统计每天的UV
        1.map  从hbase中读取一天的数据。
          以读出来的IP 为 key 任意值为value，比如 value为“1”
        2.reduce 接受map传输的数据 ，value指定为map的key。建一个list用来存储value，因为在map阶段的key是自动去重，所以list的size为访问用户的数量
    b.每天访问top10的show统计
        1.对数据按行进行正则匹配，只读取有“show”的行。通过正则匹配切分出 showid  以showid为key  “1”为value 传给reduce 
        2.reduce对showid 进行count得出 每条showid的访问数量。 按数量倒序取出前10个showid+count；
    c.留存
        1.map：对今天的数据 去重读取ip，以ip位key 1为value  在对昨天的数据去重读取ip，以ip位key，1为value 。
        2.reduce： 对 map 过来的 key 做count count为2 的IP即为留存用户 
    
    
    
    
    
    
    
    
    
    
    
    
    
    

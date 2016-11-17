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
二.hadoop安装
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
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    

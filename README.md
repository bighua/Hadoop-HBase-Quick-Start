## Hadoop&HBase Quick Start ##

 - 写在前面——关于Hadoop，HDFS，HBase...
 - 准备工作
 - Quick Start
     - Hadoop及HDFS文件系统
     - 基于HDFS的存储HBase
 - 了解更多

### 写在前面
  关于Hadoop，HDFS，HBase...

 - Hadoop: 对分布在 **计算机集群** 中的 **大型数据集合** ，使用简单的 **程序模型** 进行 **分布式处理** 的框架。
 - HDFS: 大型数据集合的分布式存储文件系统。
 - MapReduce: 可进行分布式处理计算的程序模型。
 - Hadoop YARN: 用于任务调度和集群资源管理的框架
 - HBase: Apache HBase™ is the Hadoop database, a distributed, scalable, big data store（引自官网）。基于Hadoop和hdfs的一个分布式，可扩展的大型数据存储数据库。

### 准备工作

 - 操作系统：Linux 32 bit，不太熟悉Linux的建议使用Ubuntu。
     - 因为Hadoop-2.2.0的release版本是32位的，为了避免一些不可预期的错误，建议安装32位操作系统。如果使用64位操作系统，需要做一些额外的配置，在后面的 `配置Hadoop和HDFS`中会提及。
     - 一台机器肯定谈不上分布式。以下以两台机器的集群为例，最好配置好每台机器的host，假设两台机器分别为master,slave-1。
 - JDK：1.6及以上。hadoop和hbase都是基于java开发的。
 - SSH：集群中机器之间的通信基于SSH的公私密钥。推荐网上一篇安装指南吧（http://blog.csdn.net/yxc135/article/details/8462506）
 
### Quick Start
 我们通过在安装和配置Hadoop，HBase的过程中，更加直观地理解分布式存储与计算中涉及到的一些基本概念。由于Hadoop和HBase由不同团队开发，相互支持的发布版本参见下表。![](/pic/HBase-Hadoop%20version%20support.png?raw=true)
以下介绍的安装和配置基于 **Hadoop-2.2.0** 和 **HBase-0.96.2** 
    
#### Hadoop及HDFS文件系统
 - 下载安装Hadoop
     - 下载地址：http://apache.fayea.com/apache-mirror/hadoop/common/hadoop-2.2.0/hadoop-2.2.0.tar.gz
     - 命令行中：`tar -xzvf hadoop-2.2.0.tar.gz`，解压缩到当前路径的文件夹hadoop-2.2.0下

 - 配置Hadoop和HDFS
   配置文件都在`hadoop-2.2.0/etc/hadoop/`目录下
    
     - hadoop-env.xml：找到这行注释 `# The java implementation to use.` ，然后将下面的配置贴在注释下面，`JAVA_HOME`,`HADOOP_HOME`改成自己机器jdk和hadoop的安装路径。 **在64位机器上，最后两行的配置是必须的，32位机器上可以删掉**
        
       ```
        # The java implementation to use.
        export JAVA_HOME=/usr/lib/jdk1.7.0_45
        
        export HADOOP_HOME=/home/zhuxh/hadoop-2.2.0
        export HADOOP_MAPRED_HOME=$HADOOP_HOME
        export HADOOP_COMMON_HOME=$HADOOP_HOME
        export HADOOP_HDFS_HOME=$HADOOP_HOME
        export YARN_HOME=$HADOOP_HOME
        export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
        export HADOOP_COMMON_LIB_NATIVE_DIR=${HADOOP_HOME}/lib/native
        export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
       ```

     - core-site.xml
    
       ```
        <configuration>
          <property>
            <name>fs.defaultFS</name>
            <value>hdfs://master:8010</value>
            <description>The name of the default file system. A URI whose scheme and authority determine the FileSystem implementation. The uri's scheme determines the config property (fs.SCHEME.impl) naming the FileSystem implementation class. The uri's authority is used to determine the host, port, etc. for a filesystem.
            </description>
          </property>
          <property>
           <name>hadoop.tmp.dir</name>
           <value>/home/zhuxh/hadoop-2.2.0/tmp/</value>
           <description>A base for other temporary directories.
           </description>
          </property>
        </configuration>
       ```
        
        - fs.defaultFS：配置Hadoop运行其上的文件系统，`hdfs://master:8010`指明在HDFS分布式文件系统上运行Hadoop
        - hadoop.tmp.dir：所有运行时临时文件夹的根目录，可指定为任意目录（`cd /home/zhuxh/hadoop-2.2.0 && mkdir tmp`）

     - hdfs-site.xml
     
      ```
        <configuration>
          <property>
            <name>dfs.replication</name>
            <value>1</value>
            <description>Default block replication. The actual number of replications can be specified when the file is created. The default is used if replication is not specified in create time.
            </description>
          </property>
          <property>
            <name>dfs.namenode.name.dir</name>
            <value>file:/home/zhuxh/hadoop-2.2.0/tmp/hdfs/namenode</value>
            <description>Determines where on the local filesystem the DFS name node should store the name table(fsimage). If this is a comma-delimited list of directories then the name table is replicated in all of the directories, for redundancy. 
            default: file://${hadoop.tmp.dir}/dfs/name.
            </description>
          </property>
          <property>
            <name>dfs.datanode.data.dir</name>
            <value>file:/home/zhuxh/hadoop-2.2.0/tmp/hdfs/datanode</value>
            <description>Determines where on the local filesystem an DFS data node should store its blocks. If this is a comma-delimited list of directories, then data will be stored in all named directories, typically on different devices. Directories that do not exist are ignored.
      default: file://${hadoop.tmp.dir}/dfs/data
            </description>
          </property>
        </configuration>
      ```
      
        - dfs.replication：数据需要的副本数量，这里的值设置为slave机器的数量。
        
            > 分布式数据的副本：为了满足分布式架构的可扩展性，集群中的机器可以动态的增加或减少，以满足业务的不同需求。同时为了满足系统的可用性，数据必须存有多个副本在不同的机器上，应对当某个机器当机或者网络无法连接等情况下，其他的机器仍然可以提供相同的数据。
            
        - dfs.namenode.name.dir：HDFS文件系统的nameNode节点存储数据的文件夹路径，缺省为${hadoop.tmp.dir}/dfs/name（`hadoop.tmp.dir` 是core-site.xml配置文件中定义的值）
        
        - dfs.datanode.data.dir：HDFS文件系统的dataNode节点存储数据的文件夹路径，缺省为${hadoop.tmp.dir}/dfs/data（`hadoop.tmp.dir` 是core-site.xml配置文件中定义的值）
        
            > 分布式的主从结构：HDFS文件系统架构是典型的主从结构（master-slave），其中nameNode机器节点即是master（只有一个），主要负责管理slave机器，调度来自客户端的读写等;其他的dataNode机器节点即是slave，主要用来存储数据，接受客户端读写操作，数据副本的复制等。下图比较清晰的描述了HDFS的架构![](/pic/HDFS%20Arch.png?raw=true)
        
     - mapred-site.xml
       配置目录下有一个模板文件mapred-site.xml.template，使用命令`mv mapred-site.xml.template mapred-site.xml` 改名为mapred-site.xml

       ```
       <configuration>
          <!-- real distributed -->
          <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
            <description>The runtime framework for executing MapReduce jobs. Can be one of local, classic or yarn.default is local.
            </description>
          </property>
          <property>
            <name>mapreduce.jobhistory.address</name>
            <value>master:10020</value>
            </property>
            <property>
            <name>mapreduce.jobhistory.webapp.address</name>
            <value>master:19888</value>
          </property>
          <property>
            <name>mapreduce.jobtracker.address</name>
            <value>master:54311</value>
            <description>The host and port that the MapReduce job tracker runs at.  If "local", then jobs are run in-process as a single map and reduce task.
             </description>
          </property>
          <property>
            <name>mapreduce.job.maps</name>
            <value>10</value>
            <description>The default number of map tasks per job. Ignored when mapreduce.jobtracker.address is "local".          
            </description>
          </property>
          <property>
            <name>mapreduce.job.reduces</name>
            <value>2</value>
            <description>The default number of reduce tasks per job. Typically set to 99% of the cluster's reduce capacity, so that if a node fails the reduces can still be executed in a single wave. Ignored when mapreduce.jobtracker.address is "local".
            </description>
          </property>
        </configuration>
       ```
       
          - mapreduce.framework.name：指定执行hadoop job的框架。

            > YARN(MapReduce v2): Hadoop的新一代MapReduce框架，通过[下图](http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html)简单了解一下yarn的架构如何处理分布式任务。![](/pic/yarn%20Arch.png?raw=true "yarn架构图")
              1. 当`ResourceManager`接收来自`client`的`job submission`后，会在slave节点中选取一个节点作为`App Mstr(Application Master)`。
              2. `App Mstr`会向`ResourceManager`申请task运行所需的资源，`ResourceManager`根据每个slave节点中的资源状况，以`Container`的形式在slave节点上分配每一个task运行时所需的资源，同时`App Mstr`会监控每个`Container`中task的运行状况。
              3. 在每个slave上会启动一个`Node Manager`服务，专门监控该slave上所有`Container`的资源使用情况（`Node Status`）,并定时把这些信息报告给`ResourceManager`,`ResourceManager`再根据这些信息来调度task。
               
              YARN框架中，把资源管理（内存，CPU，硬盘，网络等），任务（task）监控等功能分离到不同的组件中，以得到更好的资源利用性和可扩展性。

     - yarn-site.xml
        如果在mapred-site.xml文件中指定了`mapreduce.framework.name`的值为yarn时，该配置文件中的配置将会起作用。

        ```
        <configuration>
        <!-- Site specific YARN configuration properties -->
            <property>
              <name>yarn.resourcemanager.address</name>
              <value>master:18040</value>
            </property>
            <property>
              <name>yarn.resourcemanager.scheduler.address</name>
              <value>master:18030</value>
            </property>
            <property>
              <name>yarn.resourcemanager.webapp.address</name>
              <value>master:18088</value>
            </property>
            <property>
              <name>yarn.resourcemanager.resource-tracker.address</name>
              <value>master:18025</value>
            </property>
            <property>
              <name>yarn.resourcemanager.admin.address</name>
              <value>master:18141</value>
            </property>
           <property>
              <name>yarn.nodemanager.aux-services</name>
              <value>mapreduce_shuffle</value>
           </property>
           <property>
              <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
              <value>org.apache.hadoop.mapred.ShuffleHandler</value>
           </property>
        </configuration>
        ```
        
     - slaves
        配置slave机器的hostname，一行一个。

        ```
        slave-1
        ```
    
 - 启动hdfs和yarn
    
    以上的配置文件在master机器上配置完后，可直接拷贝到slave机器上
    
    - `hadoop-2.2.0/bin/hdfs namenode -format`，用于格式化HDFS的namenode数据的存储区
    
        > log信息：14/07/01 22:41:46 INFO common.Storage: Storage directory /home/zhuxh/hadoop-2.2.0/tmp/dfs/name has been successfully formatted.
        
        出现这段log信息时，说明format成功，存储路径即是hdfs-site.xml中配置的`dfs.namenode.name.dir` 的路径（以上信息是缺省路径）
    - `hadoop-2.2.0/sbin/start-dfs.sh`，启动hdfs的各个服务进程
         > log信息：
            Starting namenodes on [master]
            master: starting namenode, logging to             /home/zhuxh/hadoop-2.2.0/logs/hadoop-zhuxh-namenode-master.out
            slave-1: starting datanode, logging to /home/zhuxh/hadoop-2.2.0/logs/hadoop-zhuxh-datanode-slave-1.out
            Starting secondary namenodes [0.0.0.0]
            0.0.0.0: starting secondarynamenode, logging to /home/zhuxh/hadoop-2.2.0/logs/hadoop-zhuxh-secondarynamenode-master.out
        
         判断服务是否正常启动，根据上面的log信息提示到`hadoop-2.2.0/logs`查看各个服务的log信息
    - `hadoop-2.2.0/sbin/start-yarn.sh`，启动yarn的服务进程
        > log信息：
            starting yarn daemons
            starting resourcemanager, logging to /home/zhuxh/hadoop-2.2.0/logs/yarn-zhuxh-resourcemanager-master.out
            slave-1: starting nodemanager, logging to /home/zhuxh/hadoop-2.2.0/logs/yarn-zhuxh-nodemanager-slave-1.out

    - `$JAVA_HOME/bin/jps`，查看启动的服务进程（如果安装的jdk的bin目录下木有`jps`，可能安装的版本不对）
        在master机器上启动的服务

         > 29559 SecondaryNameNode
               29271 NameNode
               30041 Jps
               29789 ResourceManager
        
        在slave-1机器上启动的2个服务（slave机器上的进程是master机器通过SSH远程启动的，不需要在slave上运行上面的`start-dfs.sh`和`start-yarn.sh`命令）

         > 5161 DataNode
               5559 Jps
               5444 NodeManager
        
        或者通过`hdfs dfsadmin -report`命令查看

         > Configured Capacity: 40025579520 (37.28 GB)
                Present Capacity: 27879247872 (25.96 GB)
                DFS Remaining: 27879198720 (25.96 GB)
                DFS Used: 49152 (48 KB)
                DFS Used%: 0.00%
                Under replicated blocks: 0
                Blocks with corrupt replicas: 0
                Missing blocks: 0
                -------------------------------------------------
                Datanodes available: 1 (1 total, 0 dead)
                Live datanodes:
                Name: 192.168.234.130:50010 (slave-30)
                Hostname: slave-30
                Decommission Status : Normal
                Configured Capacity: 20079898624 (18.70 GB)
                DFS Used: 24576 (24 KB)
                Non DFS Used: 6546485248 (6.10 GB)
                DFS Remaining: 13533388800 (12.60 GB)
                DFS Used%: 0.00%
                DFS Remaining%: 67.40%
                Last contact: Tue Jul 01 23:35:48 PDT 2014

    - 监控hdfs和yarn的运行web UI
         - `http://master:50070` 分布式文件系统Web监控页面
         - `http://master:18088` yarn框架资源管理（ResourceManager）服务web监控页面
        
    -  **通过`jps`查看到服务虽然启动了，但未必没有发生错误，仍然需要通过查看log文件来确认。** 
    
 - 测试

     - 统计单词个数。Hadoop的目录下/share/hadoop/mapreduce/有一些示例程序可以在hadoop框架下运行
     
     ```
        命令行下执行：
        cd hadoop-2.2.0
        bin/hdfs dfs -mkdir /in
        bin/hdfs dfs -copyFromLocal /home/zhuxh/test /in #将/home/zhuxh目录下一个test文件复制到hdfs文件系统的in目录下
        bin/hdfs dfs -ls /in #查看in目录
        bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-2.2.0.jar wordcount /in /out #使用hadoop运行wordcount
     ```

        
        通过`http://master:18088`查看application的运行状况；![](/pic/application%20ui.png?raw=true)

        通过`http://master:50070`可以查看wordcount在hdfs文件系统的out目录下的输出结果![](/pic/hdfs-ui-1.png?raw=true)![](/pic/hdfs-ui-2.png?raw=true)
    
     - **想要知道怎么开发能在hadoop框架下运行的程序，就需要大家自己去深入研究了。。**
    
#### 基于HDFS的存储HBase


### 了解更多

- Hadoop和Hbase的设计思想来源于google三大论文——GFS（Google File System），MapReduce，BigTable

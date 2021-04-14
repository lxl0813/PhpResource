## Hbase的安装与部署

### 一、所需资源下载

hbase：http://mirrors.tuna.tsinghua.edu.cn/apache/hbase/2.2.4/hbase-2.2.4-bin.tar.gz

### 二、实验目的

1. hbase的安装与部署
3. hbase使用案例讲解

### 三、 实验步骤

### 1）hbase 安装与部署(需要配置在master和slave1)

* 将下载的hbase安装包用xftp上传到/data/program目录底下

  1.解压前先验证包是否完整

  ```
  sha256sum hbase-2.2.4-bin.tar.gz
  ```

  预计输出：

  > ec91b628352931e22a091a206be93061b6bf5364044a28fb9e82f0023aca3ca4  hbase-2.2.4-bin.tar.gz   

  2.进入/data/program目录解压hbase-2.2.4-bin.tar.gz到当前目录

  ```
  tar -xzvf hbase-2.2.4-bin.tar.gz
  ```

  3.在/data/program/目录下创建软链接hbase到/data/program/hbase-2.2.4-bin.tar.gz

  ```shell
  ln -s /data/program/hbase-2.2.4 hbase
  ```

  4.将hbase加入到环境变量中

  ```shell
  echo "export HBASE_HOME=/data/program/hbase" > /etc/profile.d/hbase.sh
  echo 'PATH=$PATH:$HBASE_HOME/bin' >> /etc/profile.d/hbase.sh
  . /etc/profile.d/hbase.sh   #让环境变量生效
  ```

  5.进入/data/program/hbase/conf/ 目录修改hbase-site.xml文件

  ```shell
  vi hbase-site.xml
  # 在<configuration>和</configuration>中间写入以下内容
        <property> 
        <name>hbase.rootdir</name> 
        <value>hdfs://master:9000/hbase/hbase_db</value>
        </property>
        <property>
        <name>hbase.unsafe.stream.capability.enforce</name>
        <value>false</value>
        </property>
        <property>
        <name>hbase.cluster.distributed</name>        #是否是分布式
        <value>true</value>
        </property>
        <property>
        <name>hbase.zookeeper.quorum</name>
        <value>master,slave1</value>                  #与你机器相对应的主机名
        </property>
        <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/data/datalake/zkdata</value>
        </property>
        <property>
        <name>hbase.master.info.port</name>
        <value>10086</value>
        </property>
  ```

  6.修改当前目录下reregionservers文件两台里的对应主机名称

  ```shell
  # 请分别修改 master 和 slave1 里的主机名称
  将masetr的reregionservers里的localhost 修改为 master
  将slave1的reregionservers里的localhost 修改为 slave1
  ```

  7.在hbase-env.sh 里添加自己的端口号（52009是我的端口号）

  ```shell
  export HBASE_SSH_OPTS="-p 52009"
  ```

  8.进入/data/bin目录 编辑hbase启动脚本（bin是软件快捷启动停止的目录）

  ```shell
  touch /data/bin/hbase-start.sh
  chmod +x /data/bin/hbase-start.sh
  vi /data/bin/hbase-start.sh
    
  # 输入如下内容：
  $HBASE_HOME/bin/start-hbase.sh
  ```

  9.进入/data/bin目录 编写hbase关闭脚本

  ```shell
  touch /data/bin/hbase-stop.sh
  chmod +x /data/bin/hbase-stop.sh
  vi /data/bin/hbase-stop.sh
    
  # 输入如下内容：
  $HBASE_HOME/bin/stop-hbase.sh
  ```

  10.master启动hbase（启动hbase之前master需启动hdfs和yarn）

  ```shell
  # 例：http://192.168.249.128:10086/ (master改为master的ip地址)
  http://master:10086查看hbase的Web UI
  ```

  ###   2）hbase 使用案例讲解

  ```sh
  # 启动代码窗口
  ./hbase shell
  1.查看表 list
  
  2.创建表 create '表名','列族1','列族2','列族N'
    例：create 'Student','StuInfo','Grades'
  
  3.添加数据(每次只能添加一个值) put '表名','行键','列族:列名','单元格的值','时间戳'
  put 'Student','0001','StuInfo:Name','Tom Green'
  put 'Student','0001','StuInfo:Age','18'
  put 'Student','0001','StuInfo:Sex','Male'
  put 'Student','0001','Grades:BigData','80'
  put 'Student','0001','Grades:Computer','90'
  put 'Student','0001','Grades:Math','85'
  put 'Student','0002','StuInfo:Name','Amy'
  put 'Student','0002','StuInfo:Age','19'
  put 'Student','0002','StuInfo:Class','01'
  put 'Student','0002','Grades:BigData','95'
  put 'Student','0002','Grades:Math','89'
  
  4.查找 get '表名','行键'
    get 'Student','0001'
  
  5.表的扫描 scan '表名'
    scan 'Student'
  
  6.令该表失效  disable '表名'
  
  7.删除行 deleteall '表名','行键'
    # 要删除表必须先让这个表失效
   
  8.删除表 drop '表名'
    disable 'Student'
    drop 'Student'
    
  9.退出 exit
  ```

  


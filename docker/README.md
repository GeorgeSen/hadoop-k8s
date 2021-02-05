# 此目录用于HDFS的Docker镜像生成

##  hadoop 基础镜像生成

 [You can get the image from the DockerHub directly.](https://registry.hub.docker.com/layers/jinbozi/hdfs/0.6/images/sha256-7cdd3c2f26d3bda7f42feae516967f4826a4e80850d63dbe69dd87fbcab0e4b6?context=explore)

> https://hub.docker.com/repository/docker/jinbozi/hdfs

If you want to build by yourself, you can refer to below command. But you should download the package of Hadoop2.9.0 and JDK1.8 from their official website, and named the package as `hadoop-2.9.0.tar.gz` and `jdk-8u11-linux-x64.tar.gz`.

加入以上两个文件的docker-build-0.6目录结构应为：

```
.
├── conf
│   ├── core-site.xml
│   ├── hdfs-site.xml
│   ├── mapred-site.xml
│   └── yarn-site.xml
├── Dockerfile
├── hadoop-2.9.0.tar.gz
├── jar
│   ├── hadoop-hdfs-2.9.0.jar
│   └── hadoop-hdfs-client-2.9.0.jar
├── jar_lib
│   ├── hadoop-hdfs-client-2.9.0.jar
│   └── jucx-1.7.0.jar
├── jdk-8u11-linux-x64.tar.gz
├── prometheus_export
│   ├── DirectorySizeCollector.class
│   ├── DirectorySizeExport.class
│   ├── DirectorySizeExport.java
│   ├── lib
│   │   ├── simpleclient-0.9.0.jar
│   │   ├── simpleclient_common-0.9.0.jar
│   │   ├── simpleclient_hotspot-0.9.0.jar
│   │   ├── simpleclient_httpserver-0.9.0.jar
│   │   └── simpleclient_pushgateway-0.9.0.jar
│   └── README.md
├── start.sh

```



```
cd docker-build-0.6
docker build -t jinbozi/hdfs:0.6
```

主要工作：

1. 利用Ubuntu基础镜像加入Hadoop源码文件以及JDK文件
2. 修改包括{HADOOP_HOME}和{JAVA_HOME}在内的环境变量
3. 其余工作已被0.1版本镜像代替，略去。。。

------



## hadoop分布式集群功能集群生成

[You can get the image from the DockerHub directly.](https://registry.hub.docker.com/layers/jinbozi/hdfs/1.0/images/sha256-b6af327900558448473286029c29e047f69ae3f3d243be494024eadc641aadeb?context=explore)

> https://hub.docker.com/repository/docker/jinbozi/hdfs

If you want to build by yourself, you can refer to below command. 

```shell
cd docker-build-0.1
docker build -t jinbozi/hdfs:0.1 .
```

### Dockerfile

主要工作：

1. 加入修改好相应名称的Hadoop配置文件{core-site.xml  hdfs-site.xml  mapred-site.xml  yarn-site.xml}
2. 加入镜像启动命令执行脚本并在容器启动时执行
3. 加入system_metrics获取脚本

### start.sh

1. 预先设置关于Hadoop，yarn，mapreduce的一些环境变量 便于之后再yaml文件中增删改查

   ```
   function setDefault(){
       if [[ "`eval echo '$'"$1"`" = "" ]]; then
           export `eval echo "$1"`=$2
       fi
   }
   
   setDefault hadoop_core_fs_defaultFS "hdfs://0.0.0.0:9000"
   setDefault hadoop_core_dfs_namenode_rpc_bind_host "0.0.0.0"
   setDefault hadoop_core_hadoop_tmp_dir "/tmp/hadoop"
   
   setDefault hadoop_hdfs_dfs_namenode_name_dir "file:///custom/hdfs_dir/namenode"
   setDefault hadoop_hdfs_dfs_datanode_data_dir "file:///custom/hdfs_dir/namenode"
   setDefault hadoop_hdfs_dfs_replication "3"
   setDefault hadoop_hdfs_dfs_datanode_balance_bandwidthPerSec "10g"
   ```

   > 以上为部分参数  

2. 将默认变量值或是yaml传入的变量值写入配置文件

   ```
   sed -i "s#hadoop_core_fs_defaultFS#${hadoop_core_fs_defaultFS}#g" $HADOOP_HOME/etc/hadoop/core-site.xml
   sed -i "s#hadoop_core_dfs_namenode_rpc-bind-host#${hadoop_core_dfs_namenode_rpc_bind_host}#g" $HADOOP_HOME/etc/hadoop/core-site.xml
   sed -i "s#hadoop_core_hadoop_tmp_dir#${hadoop_core_hadoop_tmp_dir}#g" $HADOOP_HOME/etc/hadoop/core-site.xml
   
   sed -i "s#hadoop_hdfs_dfs_namenode_name_dir#${hadoop_hdfs_dfs_namenode_name_dir}#g" $HADOOP_HOME/etc/hadoop/hdfs-site.xml
   sed -i "s#hadoop_hdfs_dfs_datanode_data_dir#${hadoop_hdfs_dfs_datanode_data_dir}#g" $HADOOP_HOME/etc/hadoop/hdfs-site.xml
   sed -i "s#hadoop_hdfs_dfs_replication#${hadoop_hdfs_dfs_replication}#g" $HADOOP_HOME/etc/hadoop/hdfs-site.xml
   sed -i "s#hadoop_hdfs_dfs_datanode_balance_bandwidthPerSec#${hadoop_hdfs_dfs_datanode_balance_bandwidthPerSec}#g" $HADOOP_HOME/etc/hadoop/hdfs-site.xml
   ```

   > 以上为部分参数

   > start.sh 部分参数具体含义可以参考以下链接：
   >
   > http://hadoop.apache.org/docs/r2.9.0/
   >
   > https://www.cnblogs.com/hatcher-h/p/13020305.html
   >

3. 根据yaml env字段注入的变量值决定启动的pod里面容器的功能 {namenode，datenode，resourcemanager，nodemanager，prometheus......}

   ```
   
   tmp=$image_function
   functions=(${tmp//|/ })  
   for function in ${functions[@]}
   do
       echo $function
       if [[ "$function" = "namenode" ]]; then
           echo "Start namenode ..."
           if [ ! -d "/custom/hdfs_dir/namenode" ]
           then
               echo "Format namenode ..."
               hdfs namenode -format
           fi
           hadoop-daemon.sh start namenode
       elif [[ "$function" = "datanode" ]]; then
           echo "Start datanode ..."
           hadoop-daemon.sh start datanode
       elif [[ "$function" = "resourcemanager" ]]; then
           echo "Start resourcemanager ..."
           yarn-daemon.sh start resourcemanager
       elif [[ "$function" = "nodemanager" ]]; then
           echo "Start nodemanager ..."
           yarn-daemon.sh start nodemanager
       elif [[ "$function" = "prometheus" ]]; then
           echo "Export prometheus metrics ..."
           java -cp /custom/prometheus_export:/custom/prometheus_export/lib/* HDFSDiskExport 10318 /custom/prometheus_export/get_system_metrics.sh /custom/hdfs_dir &
       elif [[ "$function" = "execute" ]]; then
           echo "Execute $command ..."
           $command
       elif [[ "$function" = "ssh" ]]; then
           echo "Start ssh service ..."
           /etc/init.d/ssh start
       elif [[ "$function" = "balance" ]]; then
           echo "Balance the hdfs ..."
           while true
           do
           #hadoop 自带的负载均衡命令
               hdfs balancer -threshold $hadoop_custom_balance_threshold
               sleep $hadoop_custom_balance_time_interval
           done
       fi
   done 
   
   while true; do sleep 10000; done
   ```

> 上述参数在yaml 如下字段：以balance为例：

```
apiVersion: v1
kind: Pod
metadata:
  name: hdfs-balance
  labels:
    name: "hdfs-balance"
spec:
  containers:
    - name: hdfs-balance
      image: jinbozi/hdfs:1.0
      imagePullPolicy: IfNotPresent
      securityContext:
          privileged: true
      env:
          - name: hadoop_core_fs_defaultFS
            value: "hdfs://hdfs-namenode:9000"
          - name: hadoop_yarn_yarn_resourcemanager_hostname
            value: "hdfs-namenode"
          - name: image_function
            value: "balance"
```


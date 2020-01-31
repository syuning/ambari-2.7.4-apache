# 1. 本地导入tar包格式的docker镜像/从镜像仓库icp下载镜像

## 提前安装、配置好Docker:

* 添加镜像仓库地址：

    ```vi /etc/hosts```

    > 10.10.4.181 registry.icp.com

* 更改docker配置 ```vi /usr/lib/systemd/system/docker.service``` 中的ExecStart值：

    > ExecStart=/usr/bin/dockerd --insecure-registry= registry.icp.com:5000 -H fd:// --containerd=/run/containerd/containerd.sock

* 重启docker

    ```systemctl daemon-reload```

    ```systemctl restart docker```

* 登录icp镜像仓库

    ```docker login registry.icp.com:5000```

    输入用户密码 **admin/123456a?**

* 浏览器输入 ***http://10.10.4.181:5000/harbor*** ，输入用户密码 **admin/123456a?** 登陆到仓库

## 1. 导入tar格式镜像包/下载镜像包

导入：

```docker import /path/to/编译环境.tar```

下载：

```docker pull registry.icp.com:5000/bigdata（目录）/ambari-cmp-env（镜像名称）:2.7.4.0.0（版本）```

## 2. 查看已导入的镜像的镜像ID

```docker images```

## 3. 给当前镜像打tag（下载的镜像无需执行此步）

```docker tag [镜像ID] registry.icp.com:5000/bigdata/ambari-cmp-env:2.7.4.0.0```

## 4. 以这个镜像运行一个容器

```docker run -i -t -v registry.icp.com:5000/bigdata/ambari-cmp-env:2.7.4.0.0 /bin/bash```

若想要使用本地宿主机上的本地maven库，需要在运行容器时进行挂载：

```docker run -i -t -v  /path/to/host-machine/maven-repo:/path/to/docker-image/maven-repo registry.icp.com:5000/bigdata/ambari-cmp-env-编译环境:2.7.4.0.0 /bin/bash```

默认的maven库在镜像/bigdata/maven-repo目录下，可通过maven的settings.xml配置文件查看。

# 2. 配置镜像内的环境变量

```vi /etc/profile```

镜像内已经安装的JDK、nodejs、Maven、Ant、yarn（前端）、phantomjs等，只需在profile文件内export这些环境变量：

```
export JAVA_HOME=/usr/lib/jdk-1.8.0-232
export CLASSPATH=${JAVA_HOME}/jre/lib:${JAVA_HOME}/lib
export NODE_HOME=/opt/node-v4.5.0
export PATH=${JAVA_HOME}/bin:$PATH:/opt/apache-maven-3.5.4/bin:/opt/apache-ant-1.10.7/bin:/opt/node-v4.5.0/bin:/opt/phantomjs-2.1.1-linux-x86_64/bin:/opt/phantomjs-1.9.7-linux-x86_64/bin::/opt/phantomjs-1.9.8-linux-x86_64/bin/opt/yarn-1.21.1/bin
```

# 3. 下载代码并切换至要使用的分支、执行编译

以ambari-2.7.4为例：

```cd /bigdata```

```git clone http://songyuning:Syuning1993@git.inspur.com/Insight-HD/insight-hd/ambari-2.7.4-sourcecode.git```

```cd ambari-2.7.4-sourcecode```

```git branch -a```

```git clean -d -fx ```
```git stash``` 清除git树

```git checkout -b bs remotes/origin/bs```

```git pull```

之后每次更改代码后，执行```git pull```再进行编译即可


# (可选) 4. 导出/上传docker镜像

使用```docker ps```命令查看容器ID

导出容器为镜像：
新开一个终端，执行```docker export [容器ID] > 镜像名.tar```

上传镜像：

* ```docker push registry.icp.com:5000/bigdata/ambari-cmp-env:2.7.4.0.0```

# 构建镜像 & 运行容器

构建docker镜像，--rm删除之前所有缓存，-t给生成的镜像打tag

```docker build --rm -t registry.icp.com:5000/bigdata/ambari-cmp-env:2.7.4.0.0 .```

## 查看当前所有镜像

```docker images```

## 登录、将镜像推至镜像仓库、从镜像仓库拉取镜像

```docker login```

```docker push registry.icp.com:5000/docker-images/ambari-cmp-env:2.7.4.0.0```

```docker pull registry.icp.com:5000/docker-images/ambari-cmp-env:2.7.4.0.0```

## 本地导出镜像为tar

```docker save -o latest.tar registry.icp.com:5000/docker-images/ambari-cmp-env:2.7.4.0.0```

## 本地导出容器为tar(重复执行会替换当前重名tar)

```docker export --output="exampleimage.tar" 容器ID```

```docker export 容器ID > exampleimage.tar```

## 注：导出容器时将忽略已挂载的卷，因此直接导出镜像和该镜像执行的容器可能会导出不同的tar文件

## 本地导入tar压缩包镜像
docker import /path/to/exampleimage.tar
之后docker images查看当前镜像，新生成的镜像就在列表第一个

## 以该镜像id运行一个容器，将宿主机的maven-repo挂载到镜像

```docker run -i -t -v  /Users/songyuning/workspace/Codes/maven-repo:/bigdata/maven-repo registry.icp.com:5000/bigdata/ambari-cmp-env:2.7.4.0.0 /bin/bash```

## 查看当前容器

```docker ps```

## 运行容器

```docker exec -it cf25e294d99b bash```

## 停止容器

```docker stop cf25e294d99b```

## 删除镜像

```docker rmi -f c6f5d0e1b4b3```


# 编译

## Maven设置版本准备编译

```cd /bigdata/ambari-2.7.4-sourcecode```

```mvn versions:set -DnewVersion=2.7.4.0.0```

```pushd ambari-metrics```

```mvn versions:set -DnewVersion=2.7.4.0.0```

```popd```

```pushd ambari-logsearch```

```mvn versions:set -DnewVersion=2.7.4.0.0```

```popd```

```pushd ambari-infra```

```mvn versions:set -DnewVersion=2.7.4.0.0```

```popd```

## 编译ambari

```mvn -B -X install package rpm:rpm -DnewVersion=2.7.4.0.0 -DskipTests -Dbuild-rpm -Dpython.ver="python >= 2.6" -Drat.skip=true```

## 单独编译ambari-metrics

```cd ambari-metrics```

```mvn package -Dbuild-rpm -DskipTests -Drat.skip=true```

编译过程中出错被打断时日志如下：

[ERROR] Failed to execute goal com.github.eirslett:frontend-maven-plugin:1.4:install-node-and-yarn (install node and yarn) on project ambari-web: Could not extract the Node archive: Could not extract archive: '/bigdata/maven-repo/com/github/eirslett/node/4.5.0/node-4.5.0-linux-x64.tar.gz': EOFException -> [Help 1] org.apache.maven.lifecycle.LifecycleExecutionException: Failed to execute goal com.github.eirslett:frontend-maven-plugin:1.4:install-node-and-yarn (install node and yarn) on project ambari-web: Could not extract the Node archive: Could not extract archive: '/bigdata/maven-repo/com/github/eirslett/node/4.5.0/node-4.5.0-linux-x64.tar.gz'
...
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoFailureException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :ambari-web

手动解决后使用此命令继续编译：

```mvn com.github.eirslett:frontend-maven-plugin:1.4:install-node-and-yarn -rf :ambari-web```

```
[root@57a52c01dd5e ambari-2.7.4-sourcecode]# mvn com.github.eirslett:frontend-maven-plugin:1.4:install-node-and-yarn -rf :ambari-web
Picked up _JAVA_OPTIONS: -Xmx2048m -XX:MaxPermSize=512m -Djava.awt.headless=true
OpenJDK 64-Bit Server VM warning: ignoring option MaxPermSize=512m; support was removed in 8.0
[INFO] Scanning for projects...
```

```mvn com.github.eirslett:frontend-maven-plugin:1.4:install-node-and-yarn -rf :ambari-views```


> ***【注】 在ambari-server部分源码（以及汉化后源码）的编译过程中，可能会确实部分包。需要将官网提供的rpm包解压后，复制我们编译过程中缺失的包和文件夹，再重新手动制作RPM格式的ambari-server安装包***

## 手动制作PRM包

```
[root@846ff816df7c ambari-server]# pwd
/bigdata/inspur-ambari-2.7.4-sourcecode/ambari-server/target/rpm/ambari-server
[root@846ff816df7c ambari-server]# tree
.
|-- BUILD
|-- BUILDROOT
|-- RPMS
|   `-- x86_64
|       `-- ambari-server-2.7.4.0-0.x86_64.rpm
|-- SOURCES
|-- SPECS
|   `-- ambari-server.spec
|-- SRPMS
|-- tmp-buildroot
|   `-- etc
|   `-- usr
|   `-- var

8 directories, 2 files
[root@846ff816df7c ambari-server]# 
```

```rpmbuild -bb ambari-server.spec```

# 常用命令

## 在docker容器和宿主机之间复制文件

```docker cp 1e321726a150:/bigdata/ambari-2.7.4-sourcecode/resources/maven-repo /Users/songyuning/workspace/Codes/ambari-2.7.4-sourcecode/resources/maven-repo```

```docker cp /Users/songyuning/workspace/Codes/ambari-2.7.4-sourcecode/resources/maven-reepo 1e321726a150:/bigdata/ambari-2.7.4-sourcecode/resources/maven-repo```

## rpm存放路径： 

Ambari Server - /bigdata/inspur-ambari-2.7.4-sourcecode/ambari-server/target/rpm/ambari-server/RPMS/x86_64/ambari-server-2.7.4.0-0.x86_64.rpm

Ambari Admin -  /bigdata/inspur-ambari-2.7.4-sourcecode/ambari-admin/target/rpm/ambari-admin/RPMS/noarch/ambari-admin-2.7.4.0-0.noarch.rpm

Ambari Agent - /bigdata/inspur-ambari-2.7.4-sourcecode/ambari-agent/target/rpm/ambari-agent/RPMS/x86_64/ambari-agent-2.7.4.0-0.x86_64.rpm

Ambari Funtest - /bigdata/inspur-ambari-2.7.4-sourcecode/ambari-funtest/target/rpm/ambari-funtest/RPMS/noarch/ambari-funtest-2.7.4.0-0.noarch.rpm

Ambari Infra - /bigdata/inspur-ambari-2.7.4-sourcecode/ambari-infra/target/rpm/ambari-infra/RPMS/noarch/ambari-infra-2.7.3.0-0.noarch.rpm

Ambari Logsearch - /bigdata/inspur-ambari-2.7.4-sourcecode/ambari-logsearch/target/rpm/ambari-logsearch/RPMS/noarch/ambari-logsearch-2.7.3.0-0.noarch.rpm

Ambari Metrics - /bigdata/inspur-ambari-2.7.4-sourcecode/ambari-metrics/target/rpm/ambari-metrics/RPMS/noarch/ambari-metrics-2.7.4.0-0.noarch.rpm

Ambari Project - /bigdata/inspur-ambari-2.7.4-sourcecode/ambari-project/target/rpm/ambari-project/RPMS/noarch/ambari-project-2.7.4.0-0.noarch.rpm

Ambari Server - /bigdata/inspur-ambari-2.7.4-sourcecode/ambari-server/target/rpm/ambari-server/RPMS/x86_64/ambari-server-2.7.4.0-0.x86_64.rpm

Ambari Views - /bigdata/inspur-ambari-2.7.4-sourcecode/ambari-views/target/rpm/ambari-views/RPMS/noarch/ambari-views-2.7.4.0-0.noarch.rpm

Ambari Web - /bigdata/inspur-ambari-2.7.4-sourcecode/ambari-web/target/rpm/ambari-web/RPMS/noarch/ambari-web-2.7.4.0-0.noarch.rpm

## 将rpm包拷贝至宿主机：

docker cp 6b441d6ef95f:/bigdata/ambari-2.7.4-sourcecode/ambari-server/target/rpm/ambari-server/RPMS/x86_64/ambari-server-2.7.4.0-0.x86_64.rpm `pwd`



```
[root@linux ~]# tar -cvf /tmp/etc.tar /etc         <==仅打包，不压缩！
[root@linux ~]# tar -zcvf /tmp/etc.tar.gz /etc       <==打包后，以 gzip 压缩
解压jar包 jar -xvf xxx.jar
打jar包 jar -cvfM0 xxx.jar *
```

# ambari的安装

```ambari-server setup```

配置过程中选项如下:
```
Customize user ... daemon: n
JDK: 2
JAVA_HOME:/usr/jdk64/jdk-1.8.0-232
GPL: y
database configuration: y
3 (mysql选项)
Hostname: 主机的hostname(如:manager.bigdata.com)
Port: 3306
Database name: ambari
Username: ambari
Password: bigdata
jdbc: n
输入: /usr/share/java/mysql-connector-java-5.1.48.jar
```

若在配置时没有jdbc选项，需要继续运行如下命令配置jdbc:

```ambari-server setup --jdbc-db=mysql --jdbc-driver=/usr/share/java/mysql-connector-java-5.1.48.jar```

## （重新）初始化ambari数据库

连接mysql数据库

```mysql -uroot -pbigdata```

创建ambari数据库

```create database ambari;```

创建ambari用户并赋权，```manager.bigdata.com``` 根据实际情况改为对应的hostname

```grant all privileges on *.* to 'ambari'@'manager.bigdata.com' identified by 'bigdata' with grant option;```

```FLUSH PRIVILEGES;```

```use ambari;```

```source /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql;```


/root/.ssh/id_rsa

HDP-3.0：
http://10.221.129.22/InspurHD1.0/hdp/

HDP-3.0-GPL: 
http://10.221.129.22/InspurHD1.0/hdp-gpl/

HDP-SOLR-4.0.0-400: 
http://10.221.129.22/InspurHD1.0/hdp-solr/

HDP-UTILS-1.1.0.22：
http://10.221.129.22/InspurHD1.0/hdp-utils/
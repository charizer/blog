<span id="menu">[1、kafka eagle下载](#download)</span>

[2、kafka eagle安装](#install)

[3、设置环境变量](#env)

[4、修改配置](#cfg)

[5、启动](#start)

[6、问题](#problem)

[7、使用](#use)

### <span id="download">[1、kafka eagle下载](#menu)</span>

[下载kafka-eagle-bin-1.3.2.tar.gz](http://download.kafka-eagle.org/)

### <span id="install">[2、kafka eagle安装](#menu)</span>

```
tar -zxvf kafka-eagle-bin-1.3.2.tar.gz
cd kafka-eagle-bin-1.3.2
tar -zxvf kafka-eagle-web-1.3.2-bin.tar.gz
cd kafka-eagle-web-1.3.2
```
### <span id="env">[3、设置环境变量](#menu)</span>

```
export KE_HOME=/Users/chenzr/tools/kafka-eagle-bin-1.3.2/kafka-eagle-web-1.3.2
export JAVA_HOME=/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home
export PATH=$PATH:$GOPATH/bin:$KE_HOME/bin
(mac下查看java安装位置：/usr/libexec/java_home -V)
```
### <span id="cfg">[4、修改配置](#menu)</span>

```
vim conf/system-config.properties
kafka.eagle.zk.cluster.alias=cluster1
cluster1.zk.list=127.0.0.1:2181
(单机模式)
```
参考：[kafka eagle安装部署](https://www.jianshu.com/p/552ab3e23c96)

### <span id="start">[5、启动](#menu)</span>
```
cd bin
chmod +x ke.sh
sh ke.sh
```
![image](https://github.com/chenzr/blog/blob/master/images/kafka_eagle_start.jpg)

提示web初始账户和密码分别为： Account:admin ,Password:123456
web访问：localhost:8048/ke

### <span id="problem">[6、问题](#menu)</span>

eagle正常启动，但web不能登录，查看logs的错误日志
```
tail -f ../logs/log.log
在system-config.properties添加认证及修改db配置
kafka.eagle.sasl.client=/Users/chenzr/tools/kafka-eagle-bin-1.3.2/kafka-eagle-web-1.3.2/conf
kafka.eagle.url=jdbc:sqlite:/Users/chenzr/tools/kafka-eagle-bin-1.3.2/kafka-eagle-web-1.3.2/db
```
参考：[kafka-eagle搭建中遇到的问题--web无法访问](https://blog.csdn.net/fct2001140269/article/details/83305745)

### <span id="use">[7、使用](#menu)</span>

参考：[比Kafka Mangaer更优秀的开源监控工具-Kafka Eagle](https://www.cnblogs.com/yinzhengjie/p/9957389.html)
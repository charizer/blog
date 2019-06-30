<span id="menu">[1、kafka安装](#install)</span>

[2、kafka配置](#config)

[3、启动kafka](#start)

[参考文档](#up)

### <span id="install">[1、kafka安装](#menu)</span>

```
brew install kafka
安装会依赖zookeeper。 
安装目录：/usr/local/Cellar/kafka
```

### <span id="config">[2、kafka配置](#menu)</span>

配置文件位置：
```
kafka: /usr/local/etc/kafka/server.properties (默认端口：9092)
zookeeper: /usr/local/etc/kafka/zookeeper.properties (默认端口：2181)
```

### <span id="start">[3、启动kafka](#menu)</span>
```
启动：
brew services start zookeeper
brew services start kafka
停止：
brew services stop zookeeper
brew services stop kafka
重启：
brew services restart zookeeper
brew services restart kafka
```

### <span id="up">[参考文档](#menu)</span>

[mac 安装kafka](https://www.jianshu.com/p/1f6387d18989)
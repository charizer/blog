<span id="menu">[-f 指定配置文件](#file)</span>

[-p 指定项目名称](#name)

[.env compose环境变量](#env)

[build 构建服务镜像](#build)

[config 检查配置语法](#config)

[create 创建服务容器](#create)

[down 清理项目](#down)

[exec 进入服务](#exec)

[kill 杀死服务容器](#kill)

[logs 查看服务容器日志](#logs)

[pause 暂停服务容器](#pause)

[port 查看服务容器端口状态](#port)

[ps 查看项目容器信息](#ps)

[pull 拉取项目镜像](#pull)

[push 推送项目镜像](#push)

[restart 重启服务容器](#restart)

[rm 删除服务容器](#rm)

[start 启动服务容器](#start)

[stop 停止服务容器](#stop)

[unpause 取消暂停服务容器](#unpause)

[up 启动项目](#up)

### <span id="file">[-f 指定配置文件](#menu)</span>

-f参数是用来指定 Docker Compose 的配置文件。这个参数可以使用多次，例如:
```
$ docker-compose -f docker-compose.yml -f docker-compose.admin.yml 
run backup_db
```
如果两份配置文件有同名的服务， Docker Compose 只会解析执行后面的配置文件 。 例如在 docker-compose.yml 中有一个服务叫做 webapp:
```
# docker-compose.yml 
webapp:
image: examples/web 
ports:
 - "8000:8000" 
volumes:
 - "/data"
```
在 docker-compose.admin.yml 也有一个叫做 webapp 的服务:
```
# docker-compose.admin.yml 
webapp:
 build:
 environment:
  - DEBUG=1
```
Docker Compose会执行后面的 webapp配置。 -f选项是可选的，如果不使用该选项，默认会解析当前目录下的 docker-compose.yml 文件。

### <span id="name">[-p 指定项目名称](#menu)</span>

Docker Compose 启动容器时会默认把当前的目录名称设置为容器名称前缀，例如在 Web文件夹下启动容器，配置文件中有两个服务分别是 app和 db，启动的容器名称默认是 web_db_1 和 web_app_1，如果想要指定容器项目名称(就是 web 这个前缀)，可以使用-p 参数:
```
$ docker-compose -p myapp up
myapp_db_1
myapp_app_1
```
当然如果想完全指定容器名称，可以在配置文件中设置。

### <span id="env">[.env compose环境变量](#menu)</span>

在 Docker Compose 中有一个环境配置文件 .env，这是一个隐藏文件，文件中可以设 定-些 DockerCompose 的环境变量:
```
COMPOSE_API_VERSION
COMPOSE_FILE
COMPOSE_HTTP_TIMEOUT 
COMPOSE_PROJECT_NAME 
DOCKER_CERT_PATH
DOCKER_HOST
DOCKER_TLS_VERIFY
```
• COMPOSE_PROJECT_NAME: 该变量用来定义使用 Compose 启动容器时的名称， 作用与-p 相同。
• COMPOSE_FILE:指定默认的配置文件名称，默认是 docker-compose.yml，作用类 似-f选项。
• DOCKER_HOST:指 定 Docker client 连接 Docker daemon 的地址， 默认是 unix:///var/run/docker.sock。
如果是远程操作，可以使用上面的 --tls* 选项，确保指令传输过程加密，因为有时候启动命令中会有明文密码。
```
$ cat .env
TAG=v1.5
#直接查看不会置换环境变量
$ cat docker-compose.yml
version:'3'
services:
 web:
  image:"webapp:${TAG}"
#解析时会置换.env文件中的环境变量
$ cat docker-compose config
version:'3'
services:
 web:
  image:"webapp:v1.5"
```

### <span id="build">[build 构建服务镜像](#menu)</span>
```
mysql:
 build:./db/ 
 restart: always 
 volumes:
  - /data/database:/var/lib/mysql
ui:
 build :
  context: ../
  dockerfile: myapp/ui/Dockerfile
 restart: always 
 ports:
  - "9090:80"
```
Docker Compose 提供了类似 Docker client 的构建命令，与 docker build 不同， docker-compose.yml 不只是一份启动配置，有时还包括构建定义，例如:
上面的 docker-compose.yml 中在执行 docker-compose build 命令时会自动构建 MySQL与ui两个镜像， 默认构建的镜像名称是myapp_mysql和myapp_ui。上面在设置Dockerfile 路径的同时还可以指定 Dockerfile 的上下文路径，这是 Docker client 做不到的。
在 docker-compose.yml 里面定义构建镜像时要注意把 Dockerfile 写好，因为 Docker Compose实际上是通过 docker-compose.yml读取信息解析后发给 Docker client执行的。在 docker-compose.yml 中可以使用相对路径。
在 docker-compose.yml 中通 常包含了多个容器构建、启动配置，默认情况下使用 docker-compose build 是构建 docker-compose.yml 里面的所有镜像。但是有时候只是想构建 其中 一个容器的镜像，这时候可以指定构建容器名称，例如:
```
$ docker-compose build ui
```
这个命令适用于重构部分镜像时使用 ， 避免重构全部镜像。 使用 --help 查看帮助信息会发现 ， build 命令还有 3 个选项 :
```
$ docker-compose build --help 
Build or rebuild services.
Services are built once and then tagged as 'project service’,
e .g. ’composetest_db’. If you change a service’s ’Dockerfile’ or the contents of its build directory , you can run ’ docker-compose build ’ to rebuild it.
Usage : build [options] [SERVICE .. . ]
Options:
    --force-rm Always remove intermediate containers . 
    --no-cache Do not use cache when building the image.
    --pull Always attempt to pull a newer version of the image.
```
--force-rm 和 --no-cache 都是属于在构建过程中 自动清理缓存的选项， 我们知道， 构建 过程实际是后台运行一个容器在执行 Dockerfile 的指令， 这其中会产生中间容器，使用 --force-rm 选项会在构建结束时删除这些容器。 如果构建失败时， Docker 会保留上一个临 时容器在本地，该容器保存了直至失败的指令前面的构建内容， 也就是我们说的构建缓存。 使用 --no-cache 会在构建过程中自动 删 除构建缓存 ， 上面这两 个 选项都不推荐使用。此外还有一个选项就是 --pull 选项，默认情况下 Docker Compose 会在启动容器的时候 查看本地是否有该镜像，如果有就直接使用本地己存在的镜像， 使用--pull之后即使本地有 该镜像 ， 也会执行该 pull 命令拉取镜像 ， 这样可 以确保每次启动 的容器都是基于最新的镜 像启动。

### <span id="config">[config 检查配置语法](#menu)</span>

config 命令用来检查 docker-compsoe.yml 文件是否有语法 问题，如果有会返回错误 原因。
```
$ docker-compose config --help
Validate and view the compose file .
Usage: config [options]
Options:
    -q, --quiet Only validate the configuration, don’t print anything.
    --services Print the service names , one per line .
```
例如，下面一个简单的应用有两个服务， 直接使用 config命令会输出 docker-compose.yml 的内容 :
```
$ nginx docker-compose config 
networks :
  default : 
    external:
      name: nginx_default
services : 
  app :
    container name : bbs
    image : abiosoft/caddy :php
    ports :
    - 2015:2015 
    restart : always
nginx :
  container_name: nginx 
  image: nginx :alpine 
  ports :
  - 80:80
  - 443 : 443
  restart : always
  version : ’ 2.0 ’ 
  volumes: {}
```
使用 -q 选项检查时，不会输出任何信息 ，除非有语法问题。
```
$ nginx docker-compose config -q
```
使用 --service 选项时会输出服务名称。
```
$ nginx docker-compose config --service 
acg
bbs
nginx
```

### <span id="create">[create 创建服务容器](#menu)</span>

create命令与 dockercreate类似，使用 docker-composecreate命令会创建所有服务需要 的容器 ， 但是不会运行容器。

### <span id="down">[down 清理项目](#menu)</span>

down命令与后面的 up命令相对， down命令可以停止容器并删除包括容器、网络、数 据卷等内容。 也就是只要是up命令创建的东西，使用down命令都可以删除。此外，如果 网络 、 数据卷等资源正在被其他服务使用 ， down 命令会跳过这些组件。例如 :
```
$ docker- compose down 
Stopping myapp_app_1 . .. done
Stopping myapp_db_1 . . . done
Removing myapp_app_1 . . . done
Removing myapp_db_1 ... done
Network nginx default is external , skipping
```
与 docker rm 类似 ， docker-compose down 也可以通过，-v 和--rmi 来指定删除的内容。
默认情况下， down 命令只会删除定义的服务运行的容器以及网络。通过指定 -v 参数 会删除数据卷，指定一rmi 可以删除与服务相关的镜像。 此外，使用一remove-orphans 还可以删除与服务相关，但是没有在配置文件重定义的容器。

### <span id="exec">[exec 进入服务](#menu)</span>

exec 命令与 docker exec 命令类似， 可以进入容器执行命令 ， 不同的是 docker-compose exec 后面是服务名称而不是容器名称。

### <span id="kill">[kill 杀死服务容器](#menu)</span>

使用 kill 命令 ， 默认会杀死项目下所有服务的容器。如果指定服务名称，则可以杀死 指定服务下的容器，不可以杀死指定容器名称。使用-s参数可以改变发送的信号为 SIGNAL， 默认为 SIGKILL。

### <span id="logs">[logs 查看服务容器日志](#menu)</span>

logs 命令用于查看项目日志 ， 默认这些日志包含了全部容器的日志，输出时会用不同 的颜色标示，指定服务名称可 以查看指定服务的日志。
使用--no-color可以取消颜色标示， 一般建议保留: -f可以保持输出不中断，也就是一 直显示下去，除非使用 Ctrl + C终止;-t可以显示一个时间戳在每行日志前面， 这样方便 确定日志事件发生的时间;--tail可以设定显示最后几行， 参数值的数字表示显示日志最后 的几行， 默认显示全部。

### <span id="pause">[pause 暂停服务容器](#menu)</span>
暂停项目服务可以使用 pause 命令， 默认会停止全部的服务容器的进程， 类似使用 docker pause 时的效果，如果需要停止指定的服务， 可以在后面指明服务名称。
使用了该命令的服务就像一个加锁的容器，即使使用 kill 也不能杀死 ，需要用 unpause恢复进程才可以继续对容器操作。

### <span id="port">[port 查看服务容器端口状态](#menu)</span>

在 Docker Compose 中 port 命令不如 Docker client 中的 port 命令那么灵活 ， 在 Docker Compose 中使用 port命令，不仅需要指定服务名称， 还需要指定服务容器暴露的端口 ， 才可以查看该端口在宿主机中的映射。

### <span id="ps">[ps 查看项目容器信息](#menu)</span>

ps命令用途与 docker ps类似， 使用 docker-compose ps可以查看正在运行的服务容器。该命令只有一个参数， -q 输出容器 ID，在一些脚本中非常有用。
查看项目的服务容器:
```
$ docker-compose ps
查看指定服务容器:
$ nginx docker-compose ps nginx
只显示容器ID：
$ nginx docker-compose ps -q
```

### <span id="pull">[pull 拉取项目镜像](#menu)</span>

使用 docker-compose pull 可以拉取多个镜像，因为在一份 docker-compose.yml 文件中 通常有多个服务，每个服务要有一个镜像作为镜像基础。
在 Docker Compose 所有子命令中都可以使用 -f这些参数，因此在 pull 操作时也可以指 定多个配置文件，如果服务名相同，后来者会覆盖前者， pull 操作只会拉取后出现的服务 所需要的镜像。
参数 --ignore-pull-failures 可以无视拉取失败的提示，继续执行下去，如果没有这个参 数则会在拉取失败时自动终止后面的镜像拉取操作。

### <span id="push">[push 推送项目镜像](#menu)</span>

在一些项目中，镜像井不是基于现成的 Docker镜像运行的 ， 而是在第一次启动的时 候自动创建的，因此项目中有新构建的镜像时，可以使用 push 命令推送项目的服务镜像 到仓库。

### <span id="restart">[restart 重启服务容器](#menu)</span>

在 Docker Compose 中可以像 Docker client一样操作容器， 所以当然也包括重启服务容 器， restart 默认会重启项目下的全部服务容器。

### <span id="rm">[rm 删除服务容器](#menu)</span>

使用 rm 命令当然是可以删除服务容器了 ， Docker Compose 的 rm 实际上就是对配置文 件解析后向 Docker client发送rm 的API请求， 因此Client的参数在Compose里也大都适 用，例如-v 是删除服务容器的数据卷。
-f是强制 删除服务 容器 ，与 Docker client 不同的 是， Compose 不允许强制 删除 正在运 行的容器，因此必须是停止或者杀死容器之后才能执行删除容器的操作。使用 docker-compose rm 操作时默认会提示是否真的删除容器 :
```
$ docker-compose rm
No stopped containers
$ docker-compose stop
Stopping web_app_1 .. . done 
Stopping web_db_1 ... done
$ docker-compose rm
Going to remove web_app_1 , web_db_1 Are you sure? [yN]
```
而使用-f参数之后会直接删除不会询问。

### <span id="start">[start 启动服务容器](#menu)</span>

start 是启动服务的命令，可以启动非运行的容器，默认会启动所有服务容器，指定服务名称可以启动 指定服务 。该命令使用的前提是容器己经存在。

### <span id="stop">[stop 停止服务容器](#menu)</span>

stop 显然是停止容器的命令，该命令只会停止容器，不会删除容器。默认会停止全部服务容器。可以指定服务名称来停止相应的服务。

### <span id="unpause">[unpause 取消暂停服务容器](#menu)</span>

unpause 命令在前面已经介绍过，使用 pause 命令的时候会锁定容器进程 ， 这时候需要 使用 unpause命令来取消暂停才可以继续操作容器。同样默认是针对项目全部的服务，可以指定服务名称来操作。无论是 pause还是 unpause命令，都需要容器处于运行状态才能操作。

### <span id="up">[up 启动项目](#menu)</span>

最后的 up 命令可以说是压轴出场的命令 ， 该命令与 down 命令相对 ， 使用 up 命令的 时候会从配置文件读取解析各项定义 ， 然后发给 Docker client 执行 ， up 命令可以 创 建包括 服务容器、数据卷、网络等一系列组件 ， 这也是经常使用的 Compose 命令，可以说万事从 up 开始。
Compose 的 up命令包括了构建、创建(重新创建)、启动和连接服务容器。直接使用 docker-compose up 命令可 以聚合全部 的容器信息 ， 每个容器输出内容会用不 同 的颜色区分 ， 当使用 Ctrl+C 命令停止 时， 会停止所有容器。如 果想让项 目 在后台运行 ， 就需要添 加-d 参数 ， 这样服务就会在后台运行，通过 docker-compose ps可以查看容器运行状态， logs可以看到所有容器的日志。
<span id="menu">[create 创建容器](#create)</span>

[run or start 启动容器](#run)

[-d 后台运行容器](#bg)

[--restart=always 自动重启容器](#restart)

[stop and kill 停止与杀死容器](#stop)

[rm 删除容器](#rm)

[inspect 查看容器信息](#inspect)

[attach 进入容器内部](#attach)

[exec 进入容器内部](#exec)

### <span id="create">[create 创建容器](#menu)</span>

docker create 命令用于创建一个新的容器。
```
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]

$ docker create - it ubuntu 
a4ec86af07d7leldd9lb06b334ee377989abce6b5lec323c3495063835730a2c
$ docker ps - a
CONTAINERID    IMAGE            COMMAND        CREATED       STATUS PORTS NAMES
a4ec86af07d7   ubuntu:latest    "/bin/bash"    20seconds ago Created      test
```
该命令执行的效果类似于docker run -d，即创建一个将在系统后台运行的容器。
但是与docker run -d不同的是，docker create创建的容器并未实际启动，还需要执行docker start命令或docker run命令以启动容器。
事实上，docker create命令常用于在启动容器之前进行必要的设置。

### <span id="run">[run or start 启动容器](#menu)</span>

启动容器有两种情况 ， 一种是原来没有这个容器，需要基于一个镜像启动新的容器， 另一种情况是宿主机本来有一个容器 ， 但是这个容器处于非运行状态，可以把这个非运行 的容器启动起来。启动一个新的容器可使用 docker run命令， 而启动一个已经存在的非运行状态的容器， 使用 docker start命令。
docker run 只在第一次运行时使用，将镜像放到容器中，以后再次启动这个容器时，只需要使用命令docker start 即可。
docker run相当于执行了两步操作：将镜像放入容器中（docker create）,然后将容器启动，使之变成运行时容器（docker start）。
docker start的作用是，重新启动已存在的镜像。也就是说，如果使用这个命令，我们必须事先知道这个容器的ID，或者这个容器的名字，我们可以使用docker ps找到这个容器的信息。

### <span id="bg">[-d 后台运行容器](#menu)</span>

大部分时候，运行容器都需要在后台运行较长时间， 后台运行容器需要添加 -d 参数 ， 例如在后台运行一 个 Nginx: 
```
$ docker run -d nginx : alpine
0948ef9e95091c67feclcd9712e8e2688dcd5436c816497699aee9e333791304
```
容器启动后会返回一个唯一的容器 ID， 通过 docker ps命令可以看到正在运行的容器 的基本信息。
使用 docker logs 可以查看容器的日志信息，这对于在后台运行的容器是非常重要的，有时候容器意外退出 ， 可以通过 docker logs 命令来查看容器退出的原因 :
```
$ docker logs <container id>
```

### <span id="restart">[--restart=always 自动重启容器](#menu)</span>
容器在有时候会发生一些意想不到的事情，例如应用程序内部的错误导致程序退出，从而波及容器进程，导致整个容器退出，所以需要一个办法让容器在意外退出的时候自动重启 ， 重新运行。这个参数就是一restart=always，表示一直重启 ， 参数默认值是不重启 ， docker run 使用 这个参数启动 的容器 会在容器非正常退出的时候自动重启。
```
$ docker run -d --restart=always ubuntu /bin/bash
```
这个启动命令必定是运行失败的，因为没有给 bash 提供一个输出环境， bash 启动就退 出导致容器跟着不断重启 。使用 docker ps命令可以看到容器状态一直显示:
```
Restarting (0) Less than a second ago 
```
需要注意的是这里的自动重启是面向意外退出的情况 ， 对于手动执行 docker stop 命令停止的容器 ， 属于正常退出行为 ， 并不会让容器重启。

### <span id="stop">[stop and kill 停止与杀死容器](#menu)</span>

停止容器使用的是 docker stop命令，有一个 -t 参数可以指定发送 SIGKILL信号的 时间。正常情况下 docker stop命令向容器发送的是 SIGTERM信号， 该信号会使容器正 常退出。但是有时候容器会因为各种原因对 SIGTERM 信号没有响应，这个时候设置的 -t 参数 就起了作用，当一定时间过后容器仍然没有停止就向容器发送 SIGKILL 信号 ， 让容器强制 停止。 SIGKILL 信号类似 kill命令一样会“杀”死所有正在运行的容器进程 。 例如:
```
$ docker stop <container id> 
<container id>
```
这个时候使用 docker ps -a 命令可以查看到容器状态 :
```
Exited (0) 2 seconds ago
```
退出代码是 0 时表示正常退出 。
杀死容器的操作是 docker kill，这个操作可以快速停止一个容器， 类似强制结束一个 应用 一样，这样杀死容器有可能导致数据丢失 。 例如:
```
$ docker kill <container id> 
<container id>
```
使用 docker ps -a查看容器信息可以看到退出码是 137，表示容器是非正常退出的。
```
$ docker ps - a
Exited (1 37) 1 seconds ago
```
如果是非人为的话，可以使用 docker logs查看容器日志，以便定位问题。
```
$ docker logs <container id>
```
停止所有容器。
```
$ docker kill $(docker ps -a -q) 
```
删除所有己经停止的容器 。
```
$ docker rm $(docker ps -a -q)
```
### <span id="rm">[rm 删除容器](#menu)</span>

在执行上面的停止与杀死命令之后 ，容器不会被删 除 ， 而是以停止状态保存在宿主机 此时容器不会占用磁盘之外的硬件资源。
如果用户需要释放磁盘资源， 删 除容器可 以执行 :
```
$ docker rm <container id>
<container id>
```
使用 docker rm 命令只能删除己经停止的容器， 想要删除正在运行的容器， 可以添加-f 参数，该参数会向容器发送 SIGKILL信号，例如:
```
$ docker rm -f <container id> 
<container id>
```
除了-f参数， docker rm命令还有-l与-v两个比较常用的参数。 -l参数的作用是删除容 器与其他容器的关联，但会保留容器。 -v参数可以在删除容器的时候也删除数据卷，默认 情况下容器与数据卷的生命周期是相互独立的。
```
$ docker run -v /srv:/srv -d ubuntu 
$ docker rm -v -f <container id> 
<container id>
```
### <span id="inspect">[inspect 查看容器信息](#menu)</span>

该命令用于获取容器/镜像的元数据

### <span id="attach">[attach 进入容器内部](#menu)</span>

使用 docker attach 属于 Docker 的自带命令 ， 该命令依附到正在运行的容器。
```
$docker run -dit --name test ubuntu:14.04 
lcc65411a3cb8e4d99c6f632eaf60168162ddf4a838edl5fc49717cla5f51662
$ docker attach test 
^c
root@lcc65411a3cb: /#
```
不要使用 exit命令(或者 Ctrl+ C)，那样会使 Docker容器停止。要退出容器， 可使用 Ctrl+P， 然后再使用 Ctrl+Q， 即 可退出容器的虚拟终端 ，此 时容器进程还在运行。
使用 attach 命令有 时候并不方便。当多个窗口同时 attach 到同 一个容器的时候 ， 所有 窗口都会同步显示。当某个窗口因命令阻塞时其他窗口也无法执行操作了。

### <span id="exec">[exec 进入容器内部](#menu)</span>

docker exec命令用于进入容器内部进行操作，与 attach 原理不一样， 可以像使用 SSH 登录服务器一样操作容器 。
docker exec 命令 的参数有 以下几个 。 
• -d 分离模式:在后台运行的命令 。
• -i 交互模式。
• -t 分配一个TTY。
• -u 指定用 户 和用 户组 ，格式: < name|uid >[ : < group|gid >]。
• --privileged参数会分配一个特权给 tty界面，相当于拥有了宿主机的 root权限，慎用。
下面以前面的 Ubuntu容器为例，先启动容器，然后使用 exec命令进入容器:
```
$ whoami
ubuntu
$ docker exec - i t ubuntu bash 
root@2ec3c5a455e2 : /# whoami
root
root@2ec3c5a455e2:/#
```
可以看到使用 exec 命令进入容器内部就如同进入另 一 台机器一样，可以灵活操作，并 且使用 exit 命令退出 时，不会像 attach 命令那样导致容器停止，所以非常适合在容器内部 操作。此外，每个 docker exec命令都会分配一个不同的时给用户，所以不会像 docker attach 命令那样有阻塞 。
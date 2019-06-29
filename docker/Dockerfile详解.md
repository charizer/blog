<!-- TOC -->autoauto- [FROM 指定基础镜像](#from-指定基础镜像)auto- [RUN 执行构建](#run-执行构建)auto- [ENV 设置镜像环境变量](#env-设置镜像环境变量)auto- [COPY 复制文件](#copy-复制文件)auto- [ADD 添加文件](#add-添加文件)auto- [EXPOSE 暴露指定端口](#expose-暴露指定端口)auto- [CMD 设置镜像启动命令](#cmd-设置镜像启动命令)auto- [ENTRYPOINT 设置接入点](#entrypoint-设置接入点)auto- [WORKDIR 设置工作目录](#workdir-设置工作目录)auto- [VOLUME 设置数据卷](#volume-设置数据卷)autoauto<!-- /TOC -->
## FROM 指定基础镜像

FROM 指令表示将来构建的镜像是来自哪个镜像，也就是使用哪个镜像作为基础进行构建。 一般情况下 Dockerfile 都有基础镜像， FROM 指令必须是整个 Dockerfile 的第一句有效指令。
FROM 的格式如下:
```
FROM <ImagesName : tag>
```
当同一个 Dockerfile 构建几个镜像时，可以写多个 FROM 指令，比如同时用ubuntu 和 Debian作为基础镜像构建一个系列的镜像，执行构建会使用最后一句 FROM语句。

## RUN 执行构建

这条指令用来在 Docker 的编译环境中运行指定命令 。 RUN 会在 shell 或者 exec 的环境下执行命令。
shell 格式如下:
```
RUN echo HelloW orld
```
RUN 指令会在当前镜像的顶层执行任何命令，并 commit 成新的(中间〉镜像，提交 的镜像会在后面继续用到 。
RUN 指令还有另外一种格式(exec格式〉: RUN [”程序名 ”，”参数 1”，”参数 2”]
这种格式运行程序，可以免除运行/bin/sh 的消耗 。 这种格式使用 JSON 格式将程序名 与所需参数组成-个字符串数组，所以如果参数中有引号等特殊字符，需要进行转义。
exec 格式不会触发 shell，所以$HOME 这样 的环境变量无法使用，但它可以在没有 bash的镜像中执行 ，而且可以避免错误的解析命令字符串。

## ENV 设置镜像环境变量

ENV 指令用来指定在执行 dockerrun 命令运行镜像时，自动设置的环境变量。这个环境变量可以在后续任何 RUN 指令中使用 ， 并在容器运行时保持，而且可以通过 docker run 命令的 -e 参数来修改环境变量。
ENV 指令语法如下 : 
```
ENV <key> <value>
```
使用 ENV 指令类似于 Linux 下的 export 命令， 用户可以在后续 Dockerfile 中使用这个变量。例如 :
```
ENV TARGET_DIR /app 
WORKDIR ${TARGET_DIR}
```

## COPY 复制文件

COPY 指令用来将本地的文件或文件夹复制到镜像的指定路径下 。格式如下:
```
COPY /Local/Path/File /Images/Path/File
```

## ADD 添加文件

ADD 和 COPY 作用相似，但实现不同。 ADD 指令可以从一个 URL 地址下载内容复制 到容器的文件系统中 ，还可以将压缩打包格式 的文件解开后复制到指定位置。
ADD 指令格式如下 :
```
ADD File /Images/Path/File
ADD latest.tar.gz /var/www/
```
在相同的复制命令下，使用 ADD 构建镜像的大小比 COPY 指令构建的镜像要大 ， 所以如果只是复制文件可以使用 COPY指令。

## EXPOSE 暴露指定端口

EXPOSE 指令格式如下:
```
EXPOSE <端口>[{端口}...]
```
EXPOSE 指令用于标明这个镜像中的应用将会侦昕某个端口，并且希望能将这个端口映射到主机的网络界面上。但是为了安全， docker run 命令如果没有带上响应的端口映射参数 ， Docker 并不会将端口映射出去。
此外 EXPOSE 端口是可以在多个容器之间通信用 (links)，通过 --links 参数可以让多个容器通过端口连接在一起。

## CMD 设置镜像启动命令

CMD 提供了容器默认的执行命令。 Dockerfile 只允许使用 一次 CMD 指令。使用多个 CMD会抵消之前所有的指令，只有最后一个指令生效。 一般来说这是整个Dockerfile脚本 的最后一条指令。当 Dockerfile 已经完成了所有环境的安装与配置，通过 CMD 指令来指 示 docker run 命令运行镜像时要执行的命令。格式如下 :
```
CMD [”executable”,”paraml”,”param2”]
```
值得注意的是， dockerrun命令可以覆盖CMD命令。 CMD与ENTRYPOINT的功能 极为相似。区别在于:如果dockerrun后面出现与CMD指定的相同的命令， 那么CMD会 被覆盖 : 而 ENTRYPOINT 会把容器名后面的所有内容都当成参数传递给其指定的命令(不 会对命令覆盖)。另外 ， CMD 指令还可以单独作为 ENTRYPOINT 指令的可选参数，共同 组合成一条完整的启动命令。
下面以例子说明，实验 Dockerfile 如下:
```
FROM ubuntu
CMD [”echo”,”Hello Ubuntu”]
```
然后构建镜像井运行容器 ， 运行时会返回 :
```
$ docker build -t test
Sending build context to Docker daemon 2 . 048 kB Step 1 : FROM ubuntu
---> bd3d4369aebc
Step 2 : CMD echo ’Hello Ubuntu’
---> Running in 14c9aa5280a9
---> a8391a058561
Removing intermediate container 14c9aa5280a9 Successfully built a8391a058561
$ docker run test
Hello Ubuntu
$ docker run test echo ”Hello Docker”
Hello Docker
```
当使用 docker run test echo 方式启动容器时， echo ”Hello Docker，，命令会覆盖原有 CMD 指令。也就是说 CMD 指令可以通过 docker run 命令覆盖。这一点也是 CMD 和 ENTRYPOINT 指令的最大区别。
CMD 与 RUN 的区别在于， RUN 是在 build 成镜像时就运行的，先于 C孔。和 ENTRYPOINT, CMD 会在每次启动容器的时候运行，而 RUN 只在创建镜像时执行 一 次， 固化在 image 中。

## ENTRYPOINT 设置接入点

这个指令和 CMD 很相似。 ENTRYPOINT 相当于把镜像变成一个固 定的命令工具 ，它一般是不可以通过 docker run 来改变的。而 CMD
不同， CMD 是可以通过启动命令修改内容 的 。 二者的主要区别通过实践会体会得更清晰。实验 Dockerfile 如 下 :
```
FROM ubuntu ENTRYPOINT [” echo ”]
```
实验过程如下 :
```
$ docker build -t test
Sending build context to Docker daemon 2 . 048 kB Step 1 : FROM ubuntu
---> bd3d4369aebc
Step 2 : ENTRYPOINT echo
---> Running in c6b9b6a657fd
---> aflc6cd7f531
Removing intermed工ate container c6b9b6a657fd Successfully built aflc6cd7f531
$ docker run test ”Hello Docker”
Hello Docker
```
可以看到，在 ENTRYPOINT 指令下 ， 容器就像一个 echo 程序， docker run 后续参数 就成了 echo 的参数。

## WORKDIR 设置工作目录

WORKDIR 指令指定 RUN, CMD 与 ENTRYPOINT 命令的工作目录。语法如下:
```
WORKDIR /path/to/workdir
```
同样， docker run 可以通过 -w 标志在运行时覆盖指令指定的目录。此外，可以使用多个 WORKDIR 指令 ，后续命令的参数如果是相对路径，则会基于之前命令指定的路径。例如 :
```
WORKDIR /a 
WORKDIR b 
WORKDIR c
```
则最终路径为/a/b/c。

## VOLUME 设置数据卷

VOLUME 指令用来 向基于镜像创建的容器添加数据卷(在容器中设置一个挂载点， 可 以用来让其他容器挂载或让宿主机访问，以 实现数据共享或对容器数据的备份、恢复或 迁移〉。数据卷可以在容器问共享和重用。数据卷的修改是立即生效的。数据卷的修改不 会对更新镜像产生影 响 。数据卷会一直存在 ， 直到没有任何容器使用它(没有使用它也会 在宿主机存在 ，但就不是数据卷了 ，和普通文件无异)。 VOLUME 指令在后面还会详细 介绍，这里只做简单使用说明。
VOLUME 指令格式如下 :
```
VOLUME [”/data”,”/data2”]
VOLUME /data
```
VOLUME 可以在 dockerrun 中使用 。 如果 run 命令中没有使用 ，则默认不会在宿主机 挂载这个数据卷。如果 Dockerfile 中没有设置数据卷 ， 在 docker run 中也是可以设置的。 在 Dockerfile 中声明数据卷有助于开发人员迅速定位需要保存数据的目录位置 。
下面用实例说明。写 一份 Dockerfile 如下:
```
FROM ubuntu
RUN mkdir /app && echo ”Hello” > /app/test.txt VOLUME /home
CMD [”cat”,”/app/test.txt”]
```
然后构建 :
```
$ docker build -t test
Sending build context to Docker Step 1 : FROM ubuntu
daemon
2 . 048 kB
---> bd3d4369aebc
Step 2 : RUN mkdir /app && echo
” Hello”
> /app/test . txt
---> Running in delc060fec4c
---> cef195e37ae4
Removing intermediate container delc060fec4c Step 3 : VOLUME /home
---> Running in 03256c2lc57c
---> 2625346dd697
Removing intermediate container 03256c2lc57c Step 4 : CMD cat /app/test.txt
---> Running in daac6eaffe2f
---> 75254f6ac08d
Removing intermediate container daac6eaffe2f Successfully built 75254f6ac08d
```
直接运行容器，看到返回Hello的信息 ，说明cat 命令的目标文件是容器内部的 /app/test.txt 文件。
```
$ docker run --rm test 
Hello
```
在本地创建一个文件 :
```
$ mkdir local
$ echo ”Here is test” > ~/local/test.txt
```
再运行容器，这时设置了 一个数据卷。注意，上面的 Dockerfile 并没有设置/app 为数 据卷，但是在docker run中使用了-v参数指定了/app目录， 查看cat的结果会发现目标文 件并不是容器内部的 test.txt 文件，而是宿主机上的 test.txt文件。
```
$ docker run --rm -v ~/local:/app 
test Here is not test
```
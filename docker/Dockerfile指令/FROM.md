FROM 指令表示将来构建的镜像是来自哪个镜像，也就是使用哪个镜像作为基础进行构建。 一般情况下 Dockerfile 都有基础镜像， FROM 指令必须是整个 Dockerfile 的第一句有效指令。
FROM 的格式如下:
···
FROM <ImagesName : tag>
···
当同一个 Dockerfile 构建几个镜像时，可以写多个 FROM 指令，比如同时用ubuntu 和 Debian作为基础镜像构建一个系列的镜像，执行构建会使用最后一句 FROM语句。
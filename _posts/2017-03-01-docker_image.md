# Docker/LXC 镜像制作

## 基础镜像制作

基础镜像是经由工具直接制作的镜像，也是所有服务依赖的最小镜像，一般而言，基础镜像需要完成的内容和要求有：

* 内容可控，可靠
* 容量小，仅安装的最基础的包
* 时区
* 本地化
* 源镜像地址
* 其他需要安装的软件包或agent
* 清理临时文件，日志文件等

注：不建议容器中运行ssh，如果要在容器中安装ssh服务，请在安装时请确保镜像中的ssh host key是被删除的，然后确保在容器运行时生成。

可以参考[docker官方制作脚本](https://github.com/docker/docker/tree/master/contrib) mkimage-*.sh

[mkimage-debootstrap.sh](https://github.com/docker/docker/blob/master/contrib/mkimage-debootstrap.sh)是制作debian/ubuntu的。

[mkimage-yum.sh](https://github.com/docker/docker/blob/master/contrib/mkimage-yum.sh) 是制作RHEL/Centos系统镜像的脚本。

[supermin](https://github.com/libguestfs/supermin)可以制作各种发行版的镜像。

[官方基础镜像制作文档](https://docs.docker.com/engine/userguide/eng-image/baseimages/)

## Dockerfile
[官方文档](https://docs.docker.com/engine/reference/builder/)

[最佳实践](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)

需要注意的地方有：

* 仅安装必须的软件包
* 创建时使用容易理解的标签，以便更好的管理镜像
* 固定软件版本
* 避免在Dockerfile中映射共用端口
* CMD/ENTRYPOINT命令使用数组语法
* 使用.dockerignore避免上传不需要的文件到镜像中
* 通过在行末使用\来合并ENV和RUN，减少layer的层数，避免layer超过127层
* RUN合并过多会造成镜像生成变慢，还会造成无法复用层
* 需要频繁生成镜像时，RUN可以不做合并
* 在dockerfile中将不会变动的命令放置到前面执行，时常变动的放到后面执行
* docker命令不区分大小写，但为了区别指令和指令参数，以及改善可读性，建议使用大写
* [gosu](https://github.com/tianon/gosu)
* 建议每个容器仅运行一个服务
* 慎用来历不明的镜像
* 不要在镜像中存储任何有价值的数据
* 不要用root运行进程
* 不要依赖容器中的ip
* 避免直接更新正在运行的容器，如：在容器中运行apt-get
* 仅运行有授权的镜像
* 合理使用ONBUILD
* 最后确保/var/lock/下没有遗留的文件
* 清理包管理的缓存/.git等文件
* MAINTAINER 指令已被废弃，可以用 LABEL maintainer=<...> 代替
* 避免使用COPY/ADD，如果必须使用，请写明具体的命令，如：COPY one-file.sh /somewhere/ 就比 COPY . /somewhere要好
* 避免将镜像tag命名为latest
* 构建镜像时支持用 --network 指定网络[1.13]
* docker build的--squash参数将Dockerfile中所有的操作，压缩为一层，同时保留了docker history[1.13]
* docker build的--cache-from参数，利用镜像中的 History 来判断该层是否和之前的镜像一致，从而避免重复构建[1.13]

### 从空镜像开始的Dockerfile

~~~
    FROM scratch
    MAINTAINER John <john@test.com>
    ADD rootfs.tar.xz /

    CMD ["/bin/bash"]
~~~

## 常用命令

~~~
docker build -f /path/to/a/Dockerfile .

docker build -t shykes/myapp:1.0.2 -t shykes/myapp:latest .

docker load -i nginxplus.tar
~~~

查看label

~~~
    docker inspect IMAGE_OR_CONTAINER_NAME
~~~

统计layer

~~~
    docker history Image | wc -l
~~~

## 既有镜像导出

非常不建议将现有运行的容器导出为镜像

~~~
  docker export red_panda > latest.tar
  docker export --output="latest.tar" red_panda
  
  docker save -o fedora-latest.tar fedora:latest
~~~

## 测试

[NEW-IMAGE-CHECKLIST](https://github.com/docker-library/official-images/blob/master/NEW-IMAGE-CHECKLIST.md)

[serverspec](http://serverspec.org)

[clair](https://github.com/coreos/clair)

## 参考

[official-images](https://github.com/docker-library/official-images)

## 其他

提交Dockerfile到git，利用hooks自动构建镜像

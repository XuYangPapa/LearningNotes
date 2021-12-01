# Docker基础

## 一、安装

```bash
# 1. 安装依赖包(缺少依赖包的情况下无法安装docker)
yum install -y yum-utils device-mapper-persistent-data lvm2
# 2. 配置docker软件源信息（阿里源）
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 3. 设置索引缓存加快安装速度
yum makecache fast
# 4. 安装docker
yum -y install docker-ce
# 5. 启动docker服务
systemctl start docker
# 6. 配置镜像仓库加速地址
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://kqh8****.mirror.aliyuncs.com"]
}
EOF
# 7. 重新加载服务配置文件
systemctl daemon-reload
# 8. 重启docker服务
systemctl restart docker
# 9.开机自启动
systemctl enable docker
# 10.查看docker状态
systemctl status docker
```

## 二、基本命令

### 1. 查看基本信息

```bash
# 查看本机docker相关信息
docker info
# 查看docker版本
docker version
docker -v
```

### 2. 镜像管理

```bash
# 查看本机镜像
docker images
# 搜索镜像
docker search imagename:version
# 拉取镜像
docker pull imagename:version
# 删除镜像(注意和删除容器区分)
docker rmi imagename:version
```

### 3. 容器管理-基本

```bash
# 查看正在运行的容易
docker ps
# 查看所有容器
docker ps -a
# 启动容器
docker start containername
# 停止容器
docker stop containername
# 删除容器
docker rm containername
# 查看容器详细信息
docker inspect containername
# 文件拷贝
docker cp localpath container:containerpath
```

### 4. 容器管理-重要

#### 4.1 制作容器

**`docker run [options] image [command]`**命令用于制作容器

options说明：

- **`-d, --detach=false`， 指定容器运行于前台还是后台，默认为false，退出后容器不会停止**
- `-i, --interactive=false`， 打开STDIN，用于控制台交互
- `-t, --tty=false`， 分配tty设备，该可以支持终端登录，默认为false
- `-u, --user=""`， 指定容器的用户
- `-a, --attach=[]`， 登录容器（必须是以docker run -d启动的容器）
- `-w, --workdir=""`， 指定容器的工作目录
- `-c, --cpu-shares=0`， 设置容器CPU权重，在CPU共享场景使用
- **`-e, --env=[]`， 指定环境变量，容器中可以使用该环境变量（可用多个`-e`配置多个环境变量）**
- `-m, --memory=""`， 指定容器的内存上限
- `-P, --publish-all=false`， 指定容器暴露的端口
- **`-p, --publish=[]`， 指定容器暴露的端口**
- `-h, --hostname=""`， 指定容器的主机名
- **`-v, --volume=[]`， 给容器挂载存储卷，挂载到容器的某个目录**
- `--volumes-from=[]`， 给容器挂载其他容器上的卷，挂载到容器的某个目录
- `--cap-add=[]`， 添加权限，权限清单详见：http://linux.die.net/man/7/capabilities
- `--cap-drop=[]`， 删除权限，权限清单详见：http://linux.die.net/man/7/capabilities
- `--cidfile=""`， 运行容器后，在指定文件中写入容器PID值，一种典型的监控系统用法
- `--cpuset=""`， 设置容器可以使用哪些CPU，此参数可以用来容器独占CPU
- `--device=[]`， 添加主机设备给容器，相当于设备直通
- `--dns=[]`， 指定容器的dns服务器
- `--dns-search=[]`， 指定容器的dns搜索域名，写入到容器的/etc/resolv.conf文件
- `--entrypoint=""`， 覆盖image的入口点
- `--env-file=[]`， 指定环境变量文件，文件格式为每行一个环境变量
- `--expose=[]`， 指定容器暴露的端口，即修改镜像的暴露端口
- `--link=[]`， 指定容器间的关联，使用其他容器的IP、env等信息
- `--lxc-conf=[]`， 指定容器的配置文件，只有在指定--exec-driver=lxc时使用
- **`--name=""`， 指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字**
-  -net="bridge"， 容器网络设置:
  - bridge 使用docker daemon指定的网桥
  - host //容器使用主机的网络
  - container:NAME_or_ID >//使用其他容器的网路，共享IP和PORT等网络资源
  - none 容器使用自己的网络（类似--net=bridge），但是不进行配置
- `--privileged=false`， 指定容器是否为特权容器，特权容器拥有所有的capabilities
-  --restart="no"，指定容器停止后的重启策略:
  - no：容器退出时不重启
  - on-failure：容器故障退出（返回值非零）时重启
  - always：容器退出时总是重启
- **`--rm=false`， 指定容器停止后自动删除容器(不支持以docker run -d启动的容器)**
- `--sig-proxy=true`， 设置由代理接受并处理信号，但是SIGCHLD、SIGSTOP和SIGKILL不能被代理

**常用方式**

```bash
# 已后台运行的方式基于toncat镜像启动一个名为tomcat的容器，本机8080端口映射到容器8080，本机/local/webapp文件夹挂载到容器的webapp文件夹（用于部署项目）
docker run -d -p 8080:8080 --name tomcat -v /local/webapp:/webapp tomcat:latest
# 使用镜像nginx:latest以交互模式启动一个容器,在容器内执行/bin/bash命令
docker run -it nginx:latest /bin/bash
```

#### 4.2 进入容器

当启动容器启动以后，此时想和容器进行交互就需要进入容器中，常规情况下有以下两种方式进入。

##### 4.2.1 attach方式

```bash
docker attach containerid/containername
```

注意，如果制作容器时使用了`-d`，容器会在后台以守护态运行，那么`attach`进入会直接卡住，无法操作。

如果启动命令是`docker run -it  /bin/bash`，那么在制作启动容器的当前终端就直接在容器内部，其他终端可以使用attach进入容器，但是多个终端会同步显示，并且终端exit后容器会停止运行。

##### 4.2.2 exec方式

**推荐使用**

```
docker exec -it containerid/containername /bin/bash
```

制作容器时使用的是`-d`而不是`-dit`命令，`exec -it`也可进入容器正常通过命令行操作，并且exit以后容器不会停止。




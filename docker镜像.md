

## 获取镜像：

docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]

镜像名称的格式：

Docker 镜像仓库地址：地址的格式一般是 <域名/IP>[:端口号]。默认地址是 Docker Hub。

仓库名：如之前所说，这里的仓库名是两段式名称，即 <用户名>/<软件名>。

对于 Docker Hub，如果不给出用户名，则默认为 library，也就是官方镜像。

以交互式的方式，以刚获取的镜像运行一个容器。

```bash
docker run -it --rm ubuntu bash
```

参数说明：

-it： 两个参数， -i: 交互式操作； -t： 终端； 
-rm： 容器退出后随之将其删除。   // 通常不这么用；
bash： 放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 bash。

## 列出镜像

$ docker images || docker image ls  列出已经下载的镜像；

镜像 ID 则是镜像的唯一标识，一个镜像可以对应多个标签。

## 镜像体积

本地查看时，镜像所占用空间和在 Docker Hub 上看到的镜像大小不同。(大于)

Docker Hub 中显示的体积是压缩后的体积。 而 docker image ls 显示的是镜像下载到本地后，展开的大小，准确说，是展开后的各层所占空间的总和，因为镜像到本地后，查看空间的时候，更关心的是本地磁盘空间占用的大小。

docker image ls 列表中的镜像体积总和并非是所有镜像实际硬盘消耗。由于 Docker 镜像是多层存储结构，并且可以继承、复用，因此不同镜像可能会因为使用相同的基础镜像，从而拥有共同的层。

由于 Docker 使用 Union FS，相同的层只需要保存一份即可，因此实际镜像硬盘占用空间很可能要比这个列表镜像大小的总和要小的多。

查看镜像、容器、数据卷所占用的空间：

$ docker system df


## 虚悬镜像：

镜像既没有仓库名，也没有标签，均为 <none>。

官方镜像维护时，发布了新版本后，重新 docker pull IMAGENAME:TAG时，镜像名被转移到了新下载的镜像身上，而旧的镜像上的这个名称则被取消，从而成为了 <none>

docker build 也同样可以导致这种现象。由于新旧镜像同名，旧镜像名称被取消，从而出现仓库名、标签均为 <none> 的镜像。这类无标签镜像也被称为 虚悬镜像(dangling image) 

* 查看虚悬镜像：

$ docker image ls -f dangling=true

虚悬镜像无存在价值，可以删除：  $ docker image prune

## 中间层镜像：

为了加速镜像构建、重复利用资源，Docker 会利用 中间层镜像。所以在使用一段时间后，可能会看到一些依赖的中间层镜像。

默认的 docker image ls 列表中只会显示顶层镜像，如果希望显示包括中间层镜像在内的所有镜像的话，需要加 -a 参数。

$ docker image ls -a

与虚悬镜像不同，这些无标签的镜像很多都是中间层镜像，是其它镜像所依赖的镜像。这些无标签镜像不应该删除，否则会导致上层镜像因为依赖丢失而出错。

只要删除那些依赖它们的镜像后，这些依赖的中间层镜像也会被连带删除。

中间层镜像不应该被手动删除。

## 列出部分镜像：

docker image ls 会列出所有顶级镜像，

只希望列出部分镜像：

* 根据仓库名列出镜像：

$ docker image ls ubutnu

* 列出特定的某个镜像，也就是说指定的仓库名和标签：

$ docker image ls ubuntu:18.04

* docker image ls 还支持强大的过滤器参数 --filter，或者简写 -f。

$ docker image ls -f since=mongo:3.2   // mongo:3.2 之后建立的镜像；
$ docker image ls -f before=mongo:3.2  // mongo:3.2 之前建立的镜像；

如果镜像构建时，定义了 LABEL，还可以通过 LABEL 来过滤。

$ docker image ls -f label=com.example.version=0.1

## 以特定的格式显示：

$ docker image ls -q -f dangling=true 

docker image ls 会输出一个完整的表格，但是我们并非所有时候都会需要这些内容。比如，刚才删除虚悬镜像的时候，我们需要利用 docker image ls 把所有的虚悬镜像的 ID 列出来，然后才可以交给 docker image rm 命令作为参数来删除指定的这些镜像，这个时候就用到了 -q 参数。

--filter 配合 -q 产生出指定范围的 ID 列表，然后送给另一个 docker 命令作为参数，从而针对这组实体成批的进行某种操作的做法在 Docker 命令行使用过程中非常常见，不仅仅是镜像，将来我们会在各个命令中看到这类搭配以完成很强大的功能。

可能只是对表格的结构不满意，希望自己组织列；或者不希望有标题，这样方便其它程序解析结果等，这就用到了 Go 的模板语法。

比如，下面的命令会直接列出镜像结果，并且只包含镜像ID和仓库名：

$ docker image ls --format "{{.ID}}: {{.Repository}}"

或者打算以表格等距显示，并且有标题行，和默认一样，不过自己定义列：

```bash
$ docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
IMAGE ID            REPOSITORY          TAG
47b19964fb50        ubuntu              latest
901583bfdf5a        mariadb             latest
fce289e99eb9        hello-world         latest
ff454b4a0e84        swarm               latest
```

## 删除本地镜像

* docker image rm [选项] <镜像1> [<镜像2>...]

用 ID、镜像名、摘要删除镜像:

其中，<镜像> 可以是 镜像短 ID、镜像长 ID、镜像名 或者 镜像摘要。

更多的时候是用 短 ID 来删除镜像。docker image ls 默认列出的就已经是短 ID 了，一般取前3个字符以上，只要足够区分于别的镜像就可以了。

也可以用镜像名，也就是 <仓库名>:<标签>，来删除镜像。

$ docker image rm centos:6.5

更精确的是使用 镜像摘要 删除镜像；

$ docker image ls --digests

```bash
$ docker image ls --digests 
REPOSITORY          TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
ubuntu              latest              sha256:7a47ccc3bbe8a451b500d2b53104868b46d60ee8f5b35a24b41a86077c650210   47b19964fb50        8 days ago          88.1MB
mariadb             latest              sha256:eb6acf7f599f39c42582e11b1de3866c3934da84cc45190c0aac3e8d046e4053   901583bfdf5a        3 weeks ago         367MB
hello-world         latest              sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a   fce289e99eb9        6 weeks ago         1.84kB
swarm               latest              sha256:406022f04a3d0c5ce4dbdb60422f24052c20ab7e6d41ebe5723aa649c3833975   ff454b4a0e84        8 months ago        12.7MB
```

### Untagged 和 Deleted

删除行为分为两类，一类是 Untagged，另一类是 Deleted。

使用上面命令删除镜像的时候，实际上是在要求删除某个标签的镜像。

所以首先需要做的是将满足我们要求的所有镜像标签都取消，这就是我们看到的 Untagged 的信息。

因为一个镜像可以对应多个标签，因此当我们删除了所指定的标签后，可能还有别的标签指向了这个镜像，如果是这种情况，那么 Delete 行为就不会发生。

所以并非所有的 docker image rm 都会产生删除镜像的行为，有可能仅仅是取消了某个标签而已。

当该镜像所有的标签都被取消了，该镜像很可能会失去了存在的意义，因此会触发删除行为。

镜像是多层存储结构，因此在删除的时候也是从上层向基础层方向依次进行判断删除。

镜像的多层结构让镜像复用变动非常容易，因此很有可能某个其它镜像正依赖于当前镜像的某一层。这种情况，依旧不会触发删除该层的行为。直到没有任何层依赖当前层时，才会真实的删除当前层。

容器对镜像的依赖。如果有用这个镜像启动的容器存在（即使容器没有运行），那么同样不可以删除这个镜像。


### docker image ls 命令来配合

像其它可以承接多个实体的命令一样，可以使用 docker image ls -q 来配合使用 docker image rm，这样可以成批的删除希望删除的镜像

* 删除所有仓库名为 redis 的镜像

$ docker image rm $(docker image ls -q redis)

* 或者删除mongo:3.2 之前的镜像：

$ docker image rm $(docker image ls -q -f before=mongo:3.2)

---

## 利用commit理解镜像构成

docker commit 命令除了学习之外，还有一些特殊的应用场合，比如被入侵后保存现场等。

但是，不要使用 docker commit 定制镜像，定制镜像应该使用 Dockerfile 来完成。

镜像是容器的基础，每次执行 docker run 的时候都会指定哪个镜像作为容器运行的基础。

镜像是多层存储，每一层是在前一层的基础上进行的修改；而容器同样也是多层存储，是在以镜像为基础层，在其基础上加一层作为容器运行时的存储层。

以web服务器为例子：

$ docker run --name webserver -d -p 80:80 nginx

查看正在运行的docker实例：

```bash
$ docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                NAMES
085fef9486db        nginx               "nginx -g 'daemon of…"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   webserver
```

以交互式的方式进入已经运行的container，修改文件内容后退出；

```bash
$ docker exec -it webserver bash
root@085fef9486db:/# echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
root@085fef9486db:/# exit
exit
```

* 修改了容器的文件，也就是改动了容器的存储层。我们可以通过 docker diff 命令看到具体的改动。

```bash
$ docker diff webserver 
C /root
A /root/.bash_history
C /run
A /run/nginx.pid
C /usr
C /usr/share
C /usr/share/nginx
C /usr/share/nginx/html
C /usr/share/nginx/html/index.html
C /var
C /var/cache
C /var/cache/nginx
A /var/cache/nginx/client_temp
A /var/cache/nginx/fastcgi_temp
A /var/cache/nginx/proxy_temp
A /var/cache/nginx/scgi_temp
A /var/cache/nginx/uwsgi_temp
```

当我们运行一个容器的时候（如果不使用卷的话），我们做的任何文件修改都会被记录于容器存储层里。

而 Docker 提供了一个 docker commit 命令，可以将容器的存储层保存下来成为镜像。

换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。以后我们运行这个新镜像的时候，就会拥有原有容器最后的文件变化。

docker commit 的语法格式为：

* docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]

将容器保存为镜像：

```bash
$ docker commit --author "sslinux guiyin.xiong@gmail.com" --message "修改了默认网页" webserver nginx:v2
sha256:900aa6a93dbfd0aaa41edd8321b31c5ba71b6faf9a54bbc5cbae8b8c3907b2b8

# --author 是指定修改的作者，而 --message 则是记录本次修改的内容。这点和 git 版本控制相似，不过这里这些信息可以省略留空。

$ docker image ls nginx
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               v2                  900aa6a93dbf        17 seconds ago      109MB
nginx               latest              f09fe80eb0e7        8 days ago          109MB
```

用 docker history 具体查看镜像内的历史记录，如果比较 nginx:latest 的历史记录，我们会发现新增了我们刚刚提交的这一层。

```bash
$ docker history nginx:v2
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
900aa6a93dbf        2 minutes ago       nginx -g daemon off;                            97B                 修改了默认网页
f09fe80eb0e7        8 days ago          /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  STOPSIGNAL SIGTERM           0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  EXPOSE 80                    0B                  
<missing>           8 days ago          /bin/sh -c ln -sf /dev/stdout /var/log/nginx…   22B                 
<missing>           8 days ago          /bin/sh -c set -x  && apt-get update  && apt…   53.9MB              
<missing>           8 days ago          /bin/sh -c #(nop)  ENV NJS_VERSION=1.15.8.0.…   0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  ENV NGINX_VERSION=1.15.8-…   0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B                  
<missing>           8 days ago          /bin/sh -c #(nop)  CMD ["bash"]                 0B                  
<missing>           8 days ago          /bin/sh -c #(nop) ADD file:5a6d066ba71fb0a47…   55.3MB      
```

使用刚定制的镜像，启动新的容器：

```bash
$ docker run  --name web2 -d -p 81:80 nginx:v2 
07f5bf7620a059700ec1b18a14c53498fa1ef0c4dc9fc5f18bed5e87cd394afa

$ docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
07f5bf7620a0        nginx:v2            "nginx -g 'daemon of…"   49 seconds ago      Up 47 seconds       0.0.0.0:81->80/tcp   web2
085fef9486db        nginx               "nginx -g 'daemon of…"   12 minutes ago      Up 12 minutes       0.0.0.0:80->80/tcp   webserver
```

### 慎用 docker commit

docker commit，比较直观的帮助理解镜像分层存储的概念，实际环境中并不会这样使用。

docker commit提交的形成的镜像会因为之前做的操作，导致新的镜像文件非常大。

使用 docker commit 意味着所有对镜像的操作都是黑箱操作，生成的镜像也被称为黑箱镜像(除了制作镜像的人知道执行过什么命令、怎么生成的镜像，别人根本无从得知。即使是这个制作镜像的人，过一段时间后也无法记清具体在操作的). 黑箱镜像的维护工作是非常痛苦的。

除当前层外，之前的每一层都是不会发生改变的，换句话说，任何修改的结果仅仅是在当前层进行标记、添加、修改，而不会改动上一层。如果使用 docker commit 制作镜像，以及后期修改的话，每一次修改都会让镜像更加臃肿一次，所删除的上一层的东西并不会丢失，会一直如影随形的跟着这个镜像，即使根本无法访问到。这会让镜像更加臃肿。


---

## 使用 Dockerfile 定制镜像

镜像的定制实际上就是定制每一层所添加的配置、文件。

如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

* 以nginx为例，使用Dockerfile定制镜像：

```bash
$ mkdir nginx
$ cd nginx/
$ ls
$ touch Dockerfile
$ vim Dockerfile 
$ cat Dockerfile 
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

### FROM 指定基础镜像

所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。

基础镜像是必须指定的。而 FROM 就是指定基础镜像，因此一个 Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令。

Docker Hub提供了：

* 服务类镜像
* 方便开发的各种语言应用的镜像；
* 基础的操作系统镜像；

Docker 还存在一个特殊的镜像，名为 scratch。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。

如果你以 scratch 为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。

`不以任何系统为基础，直接将可执行文件复制进镜像的做法并不罕见，比如 swarm、coreos/etcd。`
`对于 Linux 下静态编译的程序来说，并不需要有操作系统提供运行时支持，所需的一切库都已经在可执行文件里了，因此直接 FROM scratch 会让镜像体积更加小巧。`
`使用 Go 语言 开发的应用很多会使用这种方式来制作镜像，这也是为什么有人认为 Go 是特别适合容器微服务架构的语言的原因之一。`

### RUN 执行命令

RUN 指令是用来执行命令行命令的。由于命令行的强大能力，RUN 指令在定制镜像时是最常用的指令之一。其格式有两种：

* shell 格式：RUN <命令>，就像直接在命令行中输入的命令一样。刚才写的 Dockerfile 中的 RUN 指令就是这种格式。
* exec 格式：RUN ["可执行文件", "参数1", "参数2"]，这更像是函数调用中的格式。

是否就可以像 Shell 脚本一样把每个命令对应一个 RUN 呢？比如这样：

```dockerfilie
FROM debian:stretch

RUN apt-get update
RUN apt-get install -y gcc libc6-dev make wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN mkdir -p /usr/src/redis
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install
```

Dockerfile 中每一个指令都会建立一层，RUN 也不例外。每一个 RUN 的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后，commit 这一层的修改，构成新的镜像。

上面的写法，创建了 7 层镜像。这是完全没有意义的，而且很多运行时不需要的东西，都被装进了镜像里，比如编译环境、更新的软件包等等。结果就是产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。

Union FS 是有最大层数限制的，比如 AUFS，曾经是最大不得超过 42 层，现在是不得超过 127 层。

上面的 Dockerfile 正确的写法应该是这样：

```dockerfile
FROM debian:stretch

RUN buildDeps='gcc libc6-dev make wget' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

在撰写 Dockerfile 的时候，要经常提醒自己，这并不是在写 Shell 脚本，而是在定义每一层该如何构建。

Dockerfile 支持 Shell 类的行尾添加 \ 的命令换行方式，以及行首 # 进行注释的格式。良好的格式，比如换行、缩进、注释等，会让维护、排障更为容易，这是一个比较好的习惯。

这一组命令的最后添加了清理工作的命令，删除了为了编译构建所需要的软件，清理了所有下载、展开的文件，并且还清理了 apt 缓存文件。

镜像是多层存储，每一层的东西并不会在下一层被删除，会一直跟随着镜像。因此镜像构建时，一定要确保每一层只添加真正需要添加的东西，任何无关的东西都应该清理掉。

### 构建镜像：

$ docker build -t nginx:v3 .

RUN指令 启动了一个容器，并执行了所要求的命令，并最后提交了这一层，随后删除了该容器。

* docker build [选项] <上下文路径/URL/->

### 镜像构建上下文（Context）

什么是上下文呢？

Docker 在运行时分为 Docker 引擎（也就是服务端守护进程）和客户端工具。Docker 的引擎提供了一组 REST API，被称为 Docker Remote API，而如 docker 命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能。

虽然表面上我们好像是在本机执行各种 docker 功能，但实际上，一切都是使用的远程调用形式在服务端（Docker 引擎）完成。也因为这种 C/S 设计，让我们操作远程服务器的 Docker 引擎变得轻而易举。

进行镜像构建的时候，并非所有定制都会通过 RUN 指令完成，经常会需要将一些本地文件复制进镜像，比如通过 COPY 指令、ADD 指令等。而 docker build 命令构建镜像，其实并非在本地构建，而是在服务端，也就是 Docker 引擎中构建的。那么在这种客户端/服务端的架构中，如何才能让服务端获得本地文件呢？

这就引入了上下文的概念。当构建的时候，用户会指定构建镜像上下文的路径，docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。


如果在 Dockerfile 中这么写：

```dockerfile
COPY ./package.json /app/
```

这并不是要复制执行 docker build 命令所在的目录下的 package.json，也不是复制 Dockerfile 所在目录下的 package.json，而是复制 上下文（context） 目录下的 package.json。

docker build -t nginx:v3 . 中的这个 .，实际上是在指定上下文的目录，docker build 命令会将该目录下的内容打包交给 Docker 引擎以帮助构建镜像。

* 一般来说，应该会将 Dockerfile 置于一个空目录下，或者项目根目录下。如果该目录下没有所需文件，那么应该把所需文件复制一份过来。

* 如果目录下有些东西确实不希望构建时传给 Docker 引擎，那么可以用 .gitignore 一样的语法写一个 .dockerignore，该文件是用于剔除不需要作为上下文传递给 Docker 引擎的。

* Dockerfile 的文件名并不要求必须为 Dockerfile，而且并不要求必须位于上下文目录中，比如可以用 -f ../Dockerfile.php 参数指定某个文件作为 Dockerfile。

一般大家习惯性的会使用默认的文件名 Dockerfile，以及会将其置于镜像构建上下文目录中。

### 其它 docker build 的用法

* 直接用 Git repo 进行构建

docker build 还支持从 URL 构建，比如可以直接从 Git repo 中构建：

```bash
$ docker build https://github.com/twang2218/gitlab-ce-zh.git#:11.1
Sending build context to Docker daemon  5.632kB
Step 1/23 : FROM gitlab/gitlab-ce:11.1.4-ce.0 as builder
11.1.4-ce.0: Pulling from gitlab/gitlab-ce
8ee29e426c26: Pull complete 
```

这行命令指定了构建所需的 Git repo，并且指定默认的 master 分支，构建目录为 /11.1/，然后 Docker 就会自己去 git clone 这个项目、切换到指定分支、并进入到指定目录后开始构建。

* 用给定的 tar 压缩包构建

```bash
$ docker build http://server/context.tar.gz
```

如果所给出的 URL 不是个 Git repo，而是个 tar 压缩包，那么 Docker 引擎会下载这个包，并自动解压缩，以其作为上下文，开始构建。

* 从标准输入中读取 Dockerfile 进行构建

```bash
docker build - < Dockerfile

或

cat Dockerfile | docker build -
```

如果标准输入传入的是文本文件，则将其视为 Dockerfile，并开始构建。这种形式由于直接从标准输入中读取 Dockerfile 的内容，它没有上下文，因此不可以像其他方法那样可以将本地文件 COPY 进镜像之类的事情。


* 从标准输入中读取上下文压缩包进行构建

$ docker build - < context.tar.gz

如果发现标准输入的文件格式是 gzip、bzip2 以及 xz 的话，将会使其为上下文压缩包，直接将其展开，将里面视为上下文，并开始构建。


--- 

## Dockerfile 指令详解：

### COPY 复制文件

格式：

```dockerfile
COPY [--chown=<user>:<group>] <源路径>... <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]
```

和 RUN 指令一样，也有两种格式，一种类似于命令行，一种类似于函数调用。

COPY 指令将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置。

```dockerfile
COPY package.json /usr/src/app/
```

<源路径> 可以是多个，甚至可以是通配符，其通配符规则要满足 Go 的 filepath.Match 规则，如：

```dockerfile
COPY hom* /mydir/
COPY hom?.txt /mydir/
```

<目标路径> 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 WORKDIR 指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。

* WORKDIR 指定容器中的工作目录；

使用 COPY 指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等。这个特性对于镜像定制很有用。特别是构建相关文件都在使用 Git 进行管理的时候。

在使用该指令的时候还可以加上 --chown=<user>:<group> 选项来改变文件的所属用户及所属组。

```dockerfile
COPY --chown=55:mygroup files* /mydir/
COPY --chown=bin files* /mydir/
COPY --chown=1 files* /mydir/
COPY --chown=10:11 files* /mydir/
```

### ADD 更高级的复制文件

ADD 指令和 COPY 的格式和性质基本一致。

<源路径> 可以是一个 URL，这种情况下，Docker 引擎会试图去下载这个链接的文件放到 <目标路径> 去。下载后的文件权限自动设置为 600，如果这并不是想要的权限，那么还需要增加额外的一层 RUN 进行权限调整，另外，如果下载的是个压缩包，需要解压缩，也一样还需要额外的一层 RUN 指令进行解压缩。所以不如直接使用 RUN 指令，然后使用 wget 或者 curl 工具下载，处理权限、解压缩、然后清理无用文件更合理。

如果 <源路径> 为一个 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，ADD 指令将会自动解压缩这个压缩文件到 <目标路径> 去。

在某些情况下，这个自动解压缩的功能非常有用，比如官方镜像 ubuntu 中：

```dockerfile
FROM scratch
ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /
...
```

如果我们真的是希望复制个压缩文件进去，而不解压缩，这时就不可以使用 ADD 命令了，还是使用COPY。

* 尽可能的使用 COPY，因为 COPY 的语义很明确，就是复制文件而已，而 ADD 则包含了更复杂的功能，其行为也不一定很清晰。最适合使用 ADD 的场合，就是所提及的需要自动解压缩的场合。

* ADD 指令会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。

在 COPY 和 ADD 指令中选择的时候，可以遵循这样的原则，所有的文件复制均使用 COPY 指令，仅在需要自动解压缩的场合使用 ADD。


在使用该指令的时候还可以加上 --chown=<user>:<group> 选项来改变文件的所属用户及所属组。

```dockerfile
ADD --chown=55:mygroup files* /mydir/
ADD --chown=bin files* /mydir/
ADD --chown=1 files* /mydir/
ADD --chown=10:11 files* /mydir/
```

### CMD 容器启动命令:

CMD 指令的格式和 RUN 相似，也是两种格式：

* shell 格式：CMD <命令>
* exec 格式：CMD ["可执行文件", "参数1", "参数2"...]
* 参数列表格式：CMD ["参数1", "参数2"...]。在指定了 ENTRYPOINT 指令后，用 CMD 指定具体的参数。

Docker 不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。CMD 指令就是用于指定默认的容器主进程的启动命令的。

* 在运行时可以指定新的命令来替代镜像设置中的这个默认命令
* 在指令格式上，一般推荐使用 exec 格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号 "，而不要使用单引号。
* 如果使用 shell 格式的话，实际的命令会被包装为 sh -c 的参数的形式进行执行:

```dockerfile
CMD echo $HOME

#在实际执行中，会将其变更为：

CMD ["sh","-c","echo $HOME"]
# 这就是为什么我们可以使用环境变量的原因，因为这些环境变量会被 shell 进行解析处理。
```

#### 容器中应用在前台执行和后台执行的问题:

Docker 不是虚拟机，容器中的应用都应该以前台执行，而不是像虚拟机、物理机里面那样，用 upstart/systemd 去启动后台服务，容器内没有后台服务的概念。

对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。

正确的做法是直接执行 nginx 可执行文件，并且要求以前台形式运行。

```dockerfile
CMD ["nginx","-g","daemon off;"]
```

### ENTRYPOINT 入口点

ENTRYPOINT 的格式和 RUN 指令格式一样，分为 exec 格式和 shell 格式。

ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数。ENTRYPOINT 在运行时也可以替代，不过比 CMD 要略显繁琐，需要通过 docker run 的参数 --entrypoint 来指定。

当指定了 ENTRYPOINT 后，CMD 的含义就发生了改变，不再是直接的运行其命令，而是将 CMD 的内容作为参数传给 ENTRYPOINT 指令，换句话说实际执行时，将变为：

```dockerfile
<ENTRYPOINT> "<CMD>"
```

#### ENTRYPOINT的必要性：

* 场景一：让镜像变成像命令一样使用

假设我们需要一个得知自己当前公网 IP 的镜像，那么可以先用 CMD 来实现：

```dockerfile
$ vim Dockerfile 
$ cat Dockerfile 
FROM ubuntu:18.04

RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
CMD ["curl","-s","https://ip.cn"]
$ docker build -t myip .
```

如果我们需要查询当前公网 IP，只需要执行：

```bash
$ docker run myip
```

若在运行容器时，想向curl命令传递参数 -i：

执行: docker run myip -i   是肯定会报错的。

因为跟在镜像名后面的是 command，运行时会替换 CMD 的默认值。因此这里的 -i 替换了原来的 CMD，而不是添加在原来的 curl -s https://ip.cn 后面。

如果我们希望加入 -i 这参数，我们就必须重新完整的输入这个命令：

```bash
$ docker run myip curl -s https://ip.cn -i
```

使用 ENTRYPOINT 就可以解决这个问题。现在我们重新用 ENTRYPOINT 来实现这个镜像：

```dockerfile
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT [ "curl", "-s", "https://ip.cn" ]
```

再来尝试直接使用 docker run myip -i：

$ docker run myip

$ docker run myip -i

因为当存在 ENTRYPOINT 后，CMD 的内容将会作为参数传给 ENTRYPOINT，而这里 -i 就是新的 CMD，因此会作为参数传给 curl，从而达到了我们预期的效果。


* 场景二：应用运行前的准备工作

启动容器就是启动主进程，但有些时候，启动主进程前，需要一些准备工作。

比如 mysql 类的数据库，可能需要一些数据库配置、初始化的工作，这些工作要在最终的 mysql 服务器运行之前解决。

可能希望避免使用 root 用户去启动服务，从而提高安全性，而在启动服务前还需要以 root 身份执行一些必要的准备工作，最后切换到服务用户身份启动服务。或者除了服务外，其它命令依旧可以使用 root 身份执行，方便调试等。

可以写一个脚本，然后放入 ENTRYPOINT 中去执行，而这个脚本会将接到的参数（也就是 <CMD>）作为命令，在脚本最后执行。比如官方镜像 redis 中就是这么做的：


```dockerfile
$ cat Dockerfile
FROM alpine:3.4

COPY docker-entrypoint.sh /docker-entrypoint.sh
RUN ["chmod","+x","/docker-entrypoint.sh"]
ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 6379
CMD [ "redis-server" ]
```

可以看到其中为了 redis 服务创建了 redis 用户，并在最后指定了 ENTRYPOINT 为 docker-entrypoint.sh 脚本。

```bash
$ cat docker-entrypoint.sh
#!/bin/sh
...
# allow the container to be started with `--user`
if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
    chown -R redis .
    exec su-exec redis "$0" "$@"
fi

exec "$@"
```

$ docker build -t redis:custom .


该脚本的内容就是根据 CMD 的内容来判断，如果是 redis-server 的话，则切换到 redis 用户身份启动服务器，否则依旧使用 root 身份执行。比如：

$ docker run -it redis id
uid=0(root) gid=0(root) groups=0(root)




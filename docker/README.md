### 1，docker cp 容器内拷贝文件

将容器内的文件拷贝到宿主机

~~~shell
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
docker cp  96f7f14e99ab:/www /tmp/
~~~

### 2，docker commit

~~~shell
docker commit -a "username" -m "using for online speech recognition" online_kaldi_v3 online_kaldi:v1
~~~

命令使用：

docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

OPTIONS说明：

* -a :提交的镜像作者；

* -c :使用Dockerfile指令来创建镜像；

* -m :提交时的说明文字；

* -p :在commit时，将容器暂停。

### 3，本地容器打包

**Step 1：将本地容器commit成镜像**

~~~shell
docker commit -a "username"  -m "wenet_v1" 7740db56288a wenet:v1
~~~

最后两项分别为容器ID号和要生成镜像的名字和版本号

~~~~shell
docekr commit -m="提交的描述信息"-a="作者" 容器ID 要创建的目标镜像名:[标签名]
docker commit -m="vim cmd add ok" -a="tsy" 543161563fbf tsy/myubuntu:1.0
~~~~

**Step2：打包镜像**

~~~shell
docker save -o espnet_images_v2_1121.tar espnet:1103
~~~

进行打包的时候，不要使用ID e7026251937c 类似于这样，否则在load 的时候解包的时候没有名字和标签。

**step3: 上传到阿里云**

~~~shell
scp -P 22 -r 目录 host
~~~

**step4: 加载镜像**

~~~shell
docker load –i wenet_v1.tar
~~~

再通过docker images 命令查看是否加载成功，然后进行创建容器，训练模型

### 4， docker build 

~~~shell
docker build –f Dockerfile –t name_images ./
~~~

容器建立需要注意以下：

* --privileged 授予容器gdb 调试权限，放置调试报错
* -e LANG=C.UTF-8 放置vim 显示中文乱码
* -v 映射工作目录，随意。建议/opt/wenet
* **-v 数据multi，必须统一的映射 /data/speech_data/:/speech_data, 保证数据目录一直（wav.sc
* -w 进入容器后的工作目录，建议和映射的相同。
* --name 容器的名字，非必须。
* -p 将容器的端口映射处理，建议至少映射两个端口，一个22必须，一个其他自选。
* runtime 使用显卡
* 注意根据需求，选择镜像和相应tag

使用示例：

~~~
docker run --privileged -e LANG=C.UTF-8 -itd -v /data/username/wenet/:/opt/wenet -v /data/speech_data:/speech_data -w /opt/wenet --name wenet-runtime -p 1991:22 -p 19911:19911 --runtime=nvidia 10.18.210.112:40080/aivoice-tenant--aivoice-repo/wenet_base:v1.0.3
~~~

### 5，镜像目录建设

建立一个文件夹作为docker建立镜像时候的上下文，比如：

~~~shll
wenet-env-train
	Dockerfile
	requirements.txt
	src
~~~

docker默认会将该文件夹下的内容上传，但是src文件过大的时候，就会有这样的提示

~~~
Sending build context to Docker daemon 10G # 说明上下文太大了
#设置后
Sending build context to Docker daemon  7.168kB
~~~

解决办法：

**method 1**：使用 .dockerignore 文件，其内容包括，不需要上传的文件上下文，比如：

~~~
# .dockerignore 内容，其中src 被忽略
./src
~~~

问题，上下文被忽略后，还能否在COPY 进制作的镜像内。

**method 2**：新建立一个文件夹，不包括src 等大文件夹

默认交互设置 DEBIAN_FRONTEND

DEBIAN_FRONTEND 这个环境变量，告知操作系统应该从哪儿获得用户输入。如果设置为”noninteractive”，你就可以直接运行命令，而无需向用户请求输入（所有操作都是非交互式的）。这在运行apt-get命令的时候格外有用，因为它会不停的提示用户进行到了哪步并且需要不断确认。非交互模式会选择默认的选项并以最快的速度完成构建。请确保只在Dockerfile中调用的RUN命令中设置了该选项，而不是使用ENV命令进行全局的设置。因为ENV命令在整个容器运行过程中都会生效，所以当你通过BASH和容器进行交互时，如果进行了全局设置那就会出问题。

### 6，dockerfile 中设置代理

将以下地址写到 /etc/apt/apt.conf 文件下

~~~~dockerfile
Acquire::http::Proxy "http://10.18.224.XX:20024";
Acquire::https::Proxy "http://10.18.224.XX:20024";
Acquire::ftp::Proxy "http://10.18.224.XX:20024";
~~~~

### 7，FROM 拉取镜像

从镜像仓库中拉取镜像，并以此为基础镜像，开始制作。

~~~dockerfile
FROM  ubuntu:latest
~~~

### 8，ENV 设置环境变量

在制作镜像的过程中，设置环境变量。

~~~dockerfile
ENV TARGET_DIR /opt/app
WORKDIR $TARGET_DIR
~~~

也可以使用 -e 传递环境变量, 只在运行时有效。

###  9，VOLUME  添加卷 

基于镜像创建的容器添加卷,其实`VOLUME`指令只是起到了声明了容器中的目录作为匿名卷，**但是并没有将匿名卷绑定到宿主机指定目录的功能**。

~~~dockerfile
VOLUME ["/opt/project"]
~~~

在基于此镜像创建的容器中创建一个挂载点

### 10，COPY 拷贝文件到镜像内

~~~dockerfile
COPY requirement.txt /workspace/requirements
~~~

会将requirement.txt 拷贝到/workspace 目录下，并且名字叫做requirements, 当然也可以叫其他的名字。如果/workspace 目录不存在，会直接的创建。

~~~dockerfile
COPY requirement.txt /workspace/
~~~

将requirement.txt 拷贝到/workspace/ 目录下，名字不变。**注意：目标地址最后的斜杆 /**

docker 通过目的地址末尾的字符来判断文件源时目录还是文件，如果是以/ 则是目录，否则是文件。

~~~dockerfile
COPY conf/ /etc/conf/
~~~

将conf 文件夹下的文件拷贝到/etc/conf/ 文件夹下面。

### 11，ADD 拷贝并解压文件 

和COPY功能相同，但是可以自动解压文（zip 格式文件不可以）。docker 通过目的地址末尾的字符来判断文件源是目录还是文件，如果是以/ 则是文件源目录，否则是文件。

~~~dockerfile
ADD requirement.txt /workspace/requirement.txt
~~~

对于压缩文件的小魔法，将压缩文件拷贝到指定目录下，并进行解压操作。

~~~dockerfile
ADD ctc-master.tar.gz /home/
~~~

在/home/ctc-master 会有相关ctc 的解压文件。

### 12，WORKDIR 设置工作目录

~~~dockerfile
WORKDIR /workspace
~~~

设置工作目录，当进入以此镜像制作的容器后，幕默认的工作目录为 /workspace 除非在创建镜像的时候，使用 -w 进行覆盖。

### 13，RUN 运行命令

~~~dockerfile
RUN apt-get updata && apt-get install swig 
~~~

进入容器后执行，相关命令。

### 14，安装 requirement.txt

~~~dockerfile
WORKDIR ~
COPY requirement.txt .
RUN for f in $(ls requirements*.txt); do pip install --disable-pip-version-check --no-cache-dir -r $f; done
~~~


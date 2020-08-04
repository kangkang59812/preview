```bash
# -i 创建并运行 -t交互式-d后台运行    --name为容器起个名字  镜像
docker run -it --name myredis redis:latest
# 启动容器； 然后docker exec -it 6e8dcadebfda bash可进入交互式
docker start 容器名或id
# 可进入已经运行的容器, 退出容器是exit
docker attach 容器名或id 
# 从已经运行的容器中执行命令ls
docker exec -it 6e8dcadebfda ls
# 关闭 默认延时10s关闭，用来保存自己的状态  直接关闭 
docker stop -t=60 容器ID或容器名
docker kill 容器ID或容器名

# 路径映射,可以连续多个-v
docker run -it -v ./mylocal:/container redis /bin/bash
# 端口映射
# 宿主机的一个端口 只能映射到 容器上一个端口
# 容器内部的端口   可以被    宿主机多个端口  映射

# ps查看， PORTS 0.0.0.0:8088->80/tcp， 访问宿主机的 8088端口就可以即可, 指定tcp协议或udp
docker run -ti -d --name my-nginx -p 8088:80/tcp docker.io/nginx
# 查看
docker port my-nginx2 80
>> 0.0.0.8088
# 用P，随机选择可用端口，宿主机随机端口，容器内是80
docker run -ti -d --name my-nginx2 -P docker.io/nginx

# docker网络 -d 参数指定 Docker 网络类型，有 bridge、overlay, overlay 网络类型用于 Swarm mode
docker network create -d bridge test-net
docker run -itd --name test1 --network test-net ubuntu /bin/bash
#或者 对于运行中的容器test1
docker network connect test-net test1

# 从容器中打包镜像
# -a :提交的镜像作者；-c :使用Dockerfile指令来创建镜像；-m :提交时的说明文字；-p :在commit时，将容器暂停
# v1是tag
docker commit -a "runoob.com" -m "my apache" a404c6c174a2  mymysql:v1
# 删除镜像
docker rmi image_id
# 打包备份
docker save –o 打包的后的文件名.tar 镜像名
# 加载备份
docker load –i 你的备份镜像tar包

#文件拷贝，双向的
docker cp 需要拷贝的文件或目录容器名称:容器目录

#显示中文，只对当前容器的生存周期有效
> docker exec -it containerID /bin/bash
>> echo 'export LANG="en_US.UTF-8"' >> /etc/profile
>> source /etc/profile

```

#### Docker file

<img src="/Users/likangkang/Library/Application Support/typora-user-images/image-20200722171114762.png" alt="image-20200722171114762" style="zoom:35%;" />

<img src="/Users/likangkang/Library/Application Support/typora-user-images/image-20200723104601074.png" alt="image-20200723104601074" style="zoom:33%;" />

**exec 模式的特点是不会通过 shell 执行相关的命令，所以像 $HOME 这样的环境变量是取不到的**

**使用 exec 模式时，容器中的任务进程就是容器内的 1 号进程**

**使用 shell 模式时，docker 会以 /bin/sh -c "task command" 的方式执行任务命令。也就是说容器中的 1 号进程不是任务进程而是 bash 进程** CMD [ "sh", "-c", "echo $HOME" ]      CMD echo $HOME不加[]是shell模式



命令行的参数会覆盖CMD中的参数，不能传递参数

命令行的参数会追加ENTRYPOINT中的参数，ENTRYPOINT会追加CMD的参数，但CMD参数会被命令行覆盖E

https://www.cnblogs.com/sparkdev/p/8461576.html



Docker 在运行时分为 Docker引擎（服务端守护进程） 以及 客户端工具，我们日常使用各种 docker 命令，其实就是在使用客户端工具与 Docker 引擎 进行交互

docker build 命令来构建镜像时，这个构建过程其实是在 Docker引擎 中完成的，而不是在本机环境

这里就有了一个 `镜像构建上下文` 的概念，当构建的时候，由用户指定构建镜像的上下文路径，而 `docker build` 会将这个路径下所有的文件都打包上传给 `Docker 引擎`，引擎内将这些内容展开后，就能获取到所有指定上下文中的文件了

[docker中的僵尸进程是由docker-containerd-shim接管](http://shareinto.github.io/2019/01/30/docker-init(1)/)，是系统调用赋予其init(1)进程的功能（linux内核3.14以后的功能，以前是沿着其他可用线程找到最近的child_subreaper，再找到该namespace下进程号为1的进程）



https://www.cnblogs.com/sparkdev/p/7598590.html

https://www.cnblogs.com/sparkdev/p/8461576.html

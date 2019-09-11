# docker

------

> * 拉取镜像
> * docker pull ubuntu 默认最新版
> * docker pull ubuntu:latest

------

> * 查看当前有哪些镜像
> * docker images
> * 后台运行，端口映射，主机:容器
> * docker run -d -p 80:80 nginx
> * 查看当前正在运行容器有哪些
> * docke ps

------

> * 修改容器的配置，容器id前缀可唯一确定即可
> * docker exec -it 容器id bash
> * 默认nginx位置
> * usr/share/nginx

------
> * 强制删除镜像
> * docker rm -f 镜像id
> * 添加镜像
> * docker commit 镜像id 自定义镜像名

------

> * docker run -d -p 100:80 -name myname -v '~/project':/usr/share nginx:1.13
> * -d后台运行
> * -端口映射
> * -v位置映射
> * nginx:版本选择

------

##**交互式容器：**
> * docker run IMAG echo "hello world"
> * 	docker run -i -t ubuntu /bin/bash

**查看容器**
> * ocker ps			查看正在运行的容器
> * docker ps -a		查看全部容器
> * docker ps -a -l
> * docker inspect IMAG

**自定义容器名**
> * docker run --name=自定义名 -i -t IMAGE /bin/bash
> * 重新启动已停止的容器
> * docker start -i IMAGE

> * 删除停止的容器
> * docker rm 容器名

##**守护式容器：**
**1.启动守护式容器方式**
> * docker run -i -t IMAG /bin/bash
> * ctrl+p  ctrl+q

**2.启动守护式容器方式**
	附加到运行中的容器：
> * docker attach 容器名

**3.启动守护式容器方式**
> * docker run -d 镜像名 [COMMAND|ARG...]

**查看容器日志**
> * docker logs [-f][-t][--tail] 容器名
> * 	-f	--follows=true | false 默认为false	一直跟踪并返回结果
> * 	-t  --timestamps=true | false 默认为false	带时间戳
> * 	--tail="all"    返回结尾处多少数量的日志	
> * docker logs -t 10 IMAG

**查看容器内进程：**
> * docker top 容器名

**在运行中的容器内启动新进程：**
> * docker exec [-d][-i][-t] 容器名 [COMMAND][ARG...]

**停止守护式容器**
> * docker stop 容器名			发送信号让其停止
> * docker kill 容器名			直接强制瞬间停止

**使用docker帮助文件**
> * man docker-run
> * man docker-logs
> * man docker-top
> * man docker-exec

##**在容器中部署静态网站	——设置容器的端口映射**
**设置容器的端口映射**
> * run [-P][-p]
 	-P, --publish-all=true | false 默认为false
> * 		docker run -P -i -t ubuntu /bin/bash
 	-p, --publish=[]
 	    containerPort
> * 		docker run -p 80 -i -t ubuntu /bin/bash


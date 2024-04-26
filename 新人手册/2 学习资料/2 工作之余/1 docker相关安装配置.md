[TOC]



**docker和nvidia-docker区别：**

	docker不可以使用GPU的docker
	nvidia-docker可以使用GPU的docker

# 1.	脚本安装

## 1.1	安装docker

**复制以下指令一次性执行**

```shell
sudo rm -rf ~/.speedbot && git clone --depth 1 -b docker https://git.speedbot.net/devops/mirrors ~/.speedbot
chmod 777 ~/.speedbot/* && sudo ~/.speedbot/docker.sh && rm -rf ~/.speedbot
```

## 1.2	安装nvidia-docker[可选]

需要先手动配置**nvida显卡驱动**，具体见**公司平台手册 -> 常见环境配置 -> Nvidia安装**

**复制以下指令一次性执行**

```shell
sudo rm -rf ~/.speedbot && git clone --depth 1 -b nvidia-docker https://git.speedbot.net/devops/mirrors ~/.speedbot
chmod 777 ~/.speedbot/* && sudo ~/.speedbot/nvidia-docker.sh && rm -rf ~/.speedbot
```

## 1.3	安装docker-compose[可选]

**复制以下指令一次性执行**

```shell
sudo rm -rf ~/.speedbot && git clone --depth 1 -b docker-compose https://git.speedbot.net/devops/mirrors ~/.speedbot
chmod 777 ~/.speedbot/* && sudo ~/.speedbot/docker-compose.sh && rm -rf ~/.speedbot
```

# 2 更换国内源

### 2.1 docker的换源脚本

**复制以下指令一次性执行**

```shell
sudo rm -rf ~/.speedbot && git clone --depth 1 -b daemon https://git.speedbot.net/devops/mirrors ~/.speedbot
chmod 777 ~/.speedbot/* && sudo ~/.speedbot/daemon.sh && rm -rf ~/.speedbot
```

### 2.2 nvidia-docker的换源脚本

**复制以下指令一次性执行**

```shell
sudo rm -rf ~/.speedbot && git clone --depth 1 -b nvidia-daemon https://git.speedbot.net/devops/mirrors ~/.speedbot
chmod 777 ~/.speedbot/* && sudo ~/.speedbot/nvidia-daemon.sh && rm -rf ~/.speedbot
```

### 2.3 手动国内源配置文件【包含大多数国内源】

```shell
sudo vi /etc/docker/daemon.json
```

### 2.3.1 docker 源配置

```json
{
    "registry-mirrors":[
         "http://docker.mirrors.ustc.edu.cn",
         "http://hub-mirror.c.163.com",
         "http://registry.docker-cn.com",
         "http://ovfftd6p.mirror.aliyuncs.com",
         "http://registry.docker-cn.com",
         "http://docker.mirrors.ustc.edu.cn",
         "http://hub-mirror.c.163.com",
         "https://cr.console.aliyun.com"
    ] ,
    "insecure-registries":[
         "registry.docker-cn.com",
         "docker.mirrors.ustc.edu.cn",
         "mirrors.speedbot.net"
    ],
    "experimental" : true
}
```

### 2.3.2 nvidia-docker 源配置 & 默认采用nvidia-docker

```json
{
    "registry-mirrors":[
         "http://docker.mirrors.ustc.edu.cn",
         "http://hub-mirror.c.163.com",
         "http://registry.docker-cn.com",
         "http://ovfftd6p.mirror.aliyuncs.com",
         "http://registry.docker-cn.com",
         "http://docker.mirrors.ustc.edu.cn",
         "http://hub-mirror.c.163.com",
         "https://cr.console.aliyun.com"
    ] ,
    "insecure-registries":[
         "registry.docker-cn.com",
         "docker.mirrors.ustc.edu.cn",
         "mirrors.speedbot.net"
    ],
    "dns": ["223.5.5.5", "114.114.114.114"],
    "features": { "buildkit": true },
	"default-runtime": "nvidia",
    "exec-opts":["native.cgroupdriver=systemd"],
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    },
    "experimental" : true
}
```

# 3 换源后重启docker

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

若自定义源，请添加一下内容

```json
    "insecure-registries":[
         "mirrors.speedbot.net"
    ]
```

该内容是**公司镜像仓库客户端配置**，是**必须配置项**
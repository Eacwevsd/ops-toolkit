# 安装 Docker

1. 安装依赖工具

```bash
sudo apt update
sudo apt install -y ca-certificates curl
```

1. 创建用于存储 GPG 密钥的目录

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

1. 下载 GPG 密钥

```bash
sudo curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```

1. 更改密钥文件权限

```bash
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

1. 添加 Docker 的软件源

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] http://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 查看是否生成
cat /etc/apt/sources.list.d/docker.list
```

1. 更新软件源并安装 Docker

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

1. 验证 Docker 是否安装成功

```bash
docker -v
```

1. 创建 Docker 镜像加速器

```bash
sudo vim /etc/docker/daemon.json

{
  "registry-mirrors": [
    "https://1ms.run/",
    "https://docker.m.daocloud.io",
    "http://hub-mirror.c.163.com",
    "https://docker.nju.edu.cn"
  ]
}

# 重启 docker,查看镜像加速器是否生效
sudo systemctl restart docker
sudo docker info
```

1. 测试 Docker,安装成功，会输出 Hello from Docker! 的欢迎信息

```bash
sudo docker container run hello-world
```

1. 将当前用户加入 docker 组，重新登陆后执行 docker 命令就可以不用 sudo

```bash
sudo usermod -aG docker $USER
```

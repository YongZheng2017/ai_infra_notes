# Docker

## 安装Docker

```
// curl -fsSL https://get.docker.com | sh
// 官方的地址访问不了时，可以使用下面阿里云的镜像

#更新包管理工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
#添加Docker软件包源（使用keyrings方式管理GPG密钥）
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
ARCH=$(dpkg --print-architecture)
DISTRO=$(. /etc/os-release && echo "$VERSION_CODENAME")
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null <<EOF
deb [arch=${ARCH} signed-by=/etc/apt/keyrings/docker.gpg] http://mirrors.aliyun.com/docker-ce/linux/ubuntu ${DISTRO} stable
EOF
sudo apt-get update
#安装Docker社区版本，容器运行时containerd.io，以及Docker构建和Compose插件
sudo apt-get -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

参考：https://www.alibabacloud.com/help/zh/terraform/install-and-use-docker#8dca4cfa3dn0e

&nbsp;

检查:

```
docker --version    // Docker version 29.8.1, build 4a63305
docker run hello-world
```

&nbsp;

## 把用户加入 docker 组

```
sudo usermod -aG docker test
```

&nbsp;

## 修改镜像源配置

```
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors":  ["https://docker.m.daocloud.io"]
}
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker
```

&nbsp;

## 安装NVIDIA容器工具包

```
// 使用了中科大（USTC）镜像源代替了官方源
curl -fsSL https://mirrors.ustc.edu.cn/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://mirrors.ustc.edu.cn/libnvidia-container/stable/deb/amd64 /" | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit

sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

&nbsp;

测试容器内GPU访问：

```
docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi
```

&nbsp;

## 常用Docker命令

```
# List running containers
docker ps

# List all images and their sizes
docker images

# Remove unused images (reclaim disk space)
docker system prune -a

# Check GPU usage inside a running container
docker exec -it <container_id> nvidia-smi

# Copy a file from container to host
docker cp <container_id>:/workspace/results.csv ./results.csv

# View container logs
docker logs -f <container_id>
```



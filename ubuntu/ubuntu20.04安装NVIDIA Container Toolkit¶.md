# ubuntu20.04安装NVIDIA Container Toolkit

## 安装CUDA







## 一、安装docker

可以使用 Docker 的官方便捷脚本在 Ubuntu 上设置 Docker-CE：

```
curl https://get.docker.com | sh && sudo systemctl --now enable docker
```

或者可以直接apt install docker.io



## 二、设置 NVIDIA 容器工具包



### 1.设置包存储库和 GPG 密钥：

```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
      && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
      && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
            sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
            sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

gpg是一个加密工具，这里面是将nvidia官网curl过来的gpg秘钥 解码成二进制的 nvidia-container-toolkit-keyring.gpg， gpg后缀是一个密钥文件。

然后将nvidia-container-toolkit-keyring.gpg和官网获取到的libnvidia-container.list合并存放在/etc/apt/sources.list.d/nvidia-container-toolkit.list（设置包存储库）



### 2.更新包列表后安装`nvidia-docker2`包（和依赖项）：

```
sudo apt-get update
```

```
sudo apt-get install -y nvidia-docker2
```

```
sudo apt-get install -y nvidia-container-toolkit
```

配置 Docker 守护进程以识别 NVIDIA 容器运行时：

```
$ sudo nvidia-ctk runtime configure --runtime=docker
```

设置默认运行时后重启Docker守护进程完成安装：

```
sudo systemctl restart docker
```

下载完后可用cuda镜像测试

```
sudo docker run --rm --gpus all nvidia/cuda:11.6.2-base-ubuntu20.04 nvidia-smi                                                                                         
```



## 三、下载ngc示例镜像

```
docker pull nvcr.io/nvidia/tao/tao-toolkit-tf:v3.22.05-beta-api
docker pull nvcr.io/nvidia/pytorch:22.09-py3
docker pull nvcr.io/nvidia/tensorflow:22.09-tf1-py3
docker pull nvcr.io/nvidia/tensorrt:22.09-py3
pytorch
RAPIDS
TensorFlow
TensorRT
Triton
```



## 四、安装jupyter

### 1.安装pip工具

```
apt install python3-pip
```

### 2.用pip安装jupyter

```
pip install notebook
```


# deepstream_python_use

deepstream从0开始

## 1 Install
### 1.1 pull docker image
参考网址：https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_docker_containers.html
```bash
docker pull nvcr.io/nvidia/deepstream:6.4-gc-triton-devel
```
### 1.2 create container
顺便在home目录下挂载一个文件夹，便于在宿主机操作容器中的文件。
``` bash
cd
mkdir -p deepstream/file_share
```

创建一个启动容器的sh文件（或者直接复制命令到命令行窗口也行）
``` bash
cd deepstream
vim start_deepstream.sh
```
填入以下内容、修改家目录中的`username`(注意，这里环境变量DISPLAY可能不生效，下面会说)：
```bash
docker run -it \
--net=host \
--gpus all \
-e DISPLAY=$DISPLAY \
--device /dev/snd \
-v /tmp/.X11-unix/:/tmp/.X11-unix \
-v /home/<username>/deepstream/file_share/:/file_share \
--privileged=true \
--restart unless-stopped \
--log-opt max-size=20m \
--log-opt max-file=1 \
--name deepstream_6.4 \
nvcr.io/nvidia/deepstream:6.4-gc-triton-devel
```
运行：
```bash
sudo sh start_deepstream.sh
```
如果不将容器内的东西其他挂载出来，可以直接跳到1.3了。
下面是将deepstream-6.4文件夹挂载出来，方便在宿主机上操作。首先将原来容器中的内容复制出来：
``` bash
cd ..
mv deepstream-6.4 /file_share/
exit
sudo docker stop deepstream_6.4
sudo docker rm deepstream_6.4
mkdir docker_share
sudo mv file_share/deepstream-6.4 docker_share/
```
然后重新运行容器（记得修改路径）：
``` bash
sudo docker run -it \
--net=host \
--gpus all \
-e DISPLAY=$DISPLAY \
--device /dev/snd \
-v /tmp/.X11-unix/:/tmp/.X11-unix \
-v /home/<username>/deepstream/file_share/:/file_share \
-v /home/<username>/deepstream/docker_share/deepstream-6.4:/opt/nvidia/deepstream/deepstream-6.4 \
--privileged=true \
--restart unless-stopped \
--log-opt max-size=20m \
--log-opt max-file=1 \
--name deepstream_6.4 \
nvcr.io/nvidia/deepstream:6.4-gc-triton-devel
```
就可以重新进入容器了：`sudo docker exec -it deepstream_6.4 bash`
### 1.3 Change the software source
更换软件源（针对中国）
```bash
mv /etc/apt/sources.list /etc/apt/sources.list.bak
vim /etc/apt/sources.list
```
填入以下内容（USTC的源，版本Ubuntu 22.04），参考网址：[USTC源](https://mirrors.ustc.edu.cn/help/ubuntu.html)：
``` bash
# 默认注释了源码仓库，如有需要可自行取消注释
deb https://mirrors.ustc.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.ustc.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
```
或者从其他地方找也行。
更新一下
``` bash
apt-get update
apt-get upgrade
```
### 1.4 Download and build deepstream_python
参考链接：https://github.com/NVIDIA-AI-IOT/deepstream_python_apps/blob/v1.1.10/bindings/README.md
```  bash
# 1. 基础环境配置：
apt install python3-gi python3-dev python3-gst-1.0 python-gi-dev git meson \
    python3 python3-pip python3.10-dev cmake g++ build-essential libglib2.0-dev \
    libglib2.0-dev-bin libgstreamer1.0-dev libtool m4 autoconf automake libgirepository1.0-dev libcairo2-dev
# 2. 转到sources文件夹，拉取代码，并转到6.4版本：
cd sources
git clone https://github.com/NVIDIA-AI-IOT/deepstream_python_apps
cd deepstream_python_apps
git checkout v1.1.10
git log
# 3. 下载子模块，应该是这个意思（等待时间较长，耐心看网络波动）
git submodule update --init
# 4. 安装gst-python
apt-get install -y apt-transport-https ca-certificates -y
update-ca-certificates
cd 3rdparty/gstreamer/subprojects/gst-python/
meson build
meson configure
cd build
ninja
ninja install
# 5. 编译构建
cd /opt/nvidia/deepstream/deepstream-6.4/sources/deepstream_python_apps/bindings
mkdir build
cd build
cmake ..
make -j$(nproc)
# 6. wheel
pip3 install ./pyds-1.1.10-py3-none*.whl
python3 -m pip install --upgrade pip
```
# 1.5 test
``` bash
cd apps/deepstream-rtsp-in-rtsp-out
python3 deepstream_test1_rtsp_in_rtsp_out.py -i 自己的rtsp://url
apt insatall xarclock
xarclock
```







# deepstream_python_use_6.4

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
填入以下内容、修改家目录中的<username>：
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


如果不将容器内的东西挂挂载出来。

# deepstream_python_use_6.4

deepstream从0开始

## 1 Install
### 1.1 pull docker image
参考网址：https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_docker_containers.html
```bash
docker pull nvcr.io/nvidia/deepstream:6.4-gc-triton-devel
```
### 1.2 create container
```bash
docker run -it \
--net=host \
--gpus all \
-e DISPLAY=$DISPLAY \
--device /dev/snd \
-v /tmp/.X11-unix/:/tmp/.X11-unix \
-v /home/zy/deepstream/file_share/:/file_share \
--privileged=true \
--restart unless-stopped \
--log-opt max-size=20m \
--log-opt max-file=1 \
--name deepstream_6.4 \
nvcr.io/nvidia/deepstream:6.4-gc-triton-devel
```


如果你不将容器内的东西挂挂载出来。

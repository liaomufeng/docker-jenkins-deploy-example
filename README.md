# docker-jenkins-deploy-example
docker jenkins测试自动部署
# 目标服务器：安装docker→启动docker daemon进程→创建镜像→tag镜像→创建容器→push  tag镜像到私服


# Jenkins：安装Jenkins→安装插件→配置私服地址（见下图）→创建项目→拉取代码→编译、打包→创建镜像→push到私服→停止容器→移除旧容器→pull镜像→创建容器→启动容器
        
==================================

# 1、安装docker，并启动docker daemon守护进程：

     systemctl daemon-reload
# 2、搭建Docker registry镜像私服（可本机也可远程机器）：

docker run -d --name docker-private-images \
-p 5000:5000 \
--restart=always \
-v ~/docker/docker-private-images:/var/lib/registry \
registry
    查看镜像私服所有镜像

curl http://镜像私服IP:5000/v2/_catalog
# 3、在Docker镜像私服的机器中，修改/etc/docker/daemon.json文件，然后重启docker daemon：


#PRIVATE_REGISTRY也可用IP地址
{
  "insecure-registries":["PRIVATE_REGISTRY:5000"] 
}
     systemctl daemon-reload
# 4、本地机器上根据项目实际名称，构建镜像（需要Dockerfile）:

docker build -t dev_docker_jar_image .
# 5、构建并运行容器：
docker run --rm dev_docker_jar_container
    参数说明：
    --rm：对于foreground容器，由于其只是在开发调试过程中短期运行，其用户数据并无保留的必要，因而可以在容器启动时可以设置--rm选项，这样在容器退出时就能够自动清理容器内部的文件系统。不能与-d同时使用，即只能自动清理foreground容器，不能自动清理detached容器。
# 6、tag镜像（push到私服，需要先tag，否则就是默认push到Docker Hub ）：
docker tag dev_docker_jar_image PRIVATE_REGISTRY:5000/dev_docker_jar_image
# 7、push镜像到私服：
docker push PRIVATE_REGISTRY:5000/dev_docker_jar_image
# 8、查看私服上所有镜像：
curl http://镜像私服IP:5000/v2/_catalog
# 9、停止容器、移除容器、移出旧镜像、pull新镜像、创建容器、启动容器：
docker stop dev_docker_jar_container
docker rm dev_docker_jar_container
docker rmi dev_docker_jar_image
docker pull PRIVATE_REGISTRY:5000/dev_docker_jar_image
docker run -d --name dev_docker_jar_container \ PRIVATE_REGISTRY:5000/dev_docker_jar_container

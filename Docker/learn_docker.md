# Docker用法学习

[toc]



### 基本概念

- Container-Images-Registries

  ![](imgs/基本概念.png)

Registry包含多个Image，一个Image可实例化为一到多个Container，多个Container运行在一台服务器中。

### 使用Docker镜像

#### 容器操作指令

```bash
docker run image_name [command] #实例化镜像并运行指令，如果镜像在本地不存在则联网下载
docker ps [-a] #列出正在运行的镜像，[-a]选项将停止的镜像也列出
docker logs container_name/id #检索容器日志
docker inspect container_name/id #得到容器的详细信息
docker stop container_name/id
docker rm ... 
docker container prune -f #强制删除所有未工作的容器
```

#### 运行服务器式的容器

- [x] 容器最好保持为无状态的，因为这样有助于更好的扩容和恢复：防止数据泄露、不一致

- 一个服务器式的容器，应该满足：
  - 长期运行
  - 监听输入的网络连接的信息

```bash
docker run -d alpine ping www.docker.com #-d选项使得容器在后台长期运行
docker logs [--since 10s] 789b #--since选项，查看最近10s内生成的日志信息
docker run -d -p 8085:80 nginx #-p选项，指定宿主主机和容器内端口的映射关系
```

**更多[参考](https://www.cnblogs.com/kevingrace/p/9453987.html)**

#### 使用Volumes

```bash
docker run -v /your/dir:/var/lib/mysql -d mysql:5.7 
```

上述指令将mysql容器在其`/var/lib/mysql`中存储的数据映射到宿主主机的目录`/your/dir/`下持久保存

#### 镜像来自何处？

![](imgs/拉取镜像.png)

```
<repository_name>/<name>:<tag>
```

- 当一个镜像被发布到一个注册表时，它的名字格式如上所示：
  - tag可选，不存在时默认是latest
  - repository_name：a registry DNS or the name of a registry in the Docker Hub

- 使用`docker pull`来手动拉取一个镜像到本地
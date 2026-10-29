# docker 创建并启动服务容器

```bash
docker run [选项] 镜像 [命令]
```

选项

- `-d`：后台运行
- `-it`：交互式终端，常用于进入容器
- `--name`：指定容器名
- `-p 宿主机端口：容器端口`：端口映射
- `-P`：随机映射容器暴露的端口
- `-v 宿主机路径：容器路径`：挂载数据卷
- `--mount`：更明确的挂载方式
- `-e KEY=VALUE`：设置环境变量
- `--env-file`：从文件读取环境变量
- `--network`：指定网络
- `--restart`：重启策略，值有 `no`、`on-failure`、`always`、`unless-stopped`
- `--rm`：容器停止后自动删除
- `-m`/`--memory`：内存限制
- `--cpus`：CPU 限制
- `-u`：指定用户
- `-w`：工作目录
- `--entrypoint`：覆盖镜像的 `ENTRYPOINT`
- `--privileged`：特权模式（慎用）

```bash
# 运行 MySQL，设置密码和数据库
docker run -d --name camps_mysql \
	-e MYSQL_ROOT_PASSWORD=root@root \
	-e MYSQL_DATABASE=bitstorm-svr \
	-p 8086:3306 \
	mysql:8.0

# 运行 redis，映射 6379 端口
docker run -d --name redis -e 6379:6379 redis

# 运行 nacos
%%/%%
```

# `docker ps` 列出容器

```bash
docker ps                          # 正在运行的容器
docker ps -a                       # 列出所有容器（包括已经停止的）
docker ps -q                       # 只显示 ID
docker ps -s                       # 显示容器大小
docker ps --filter "status=exited"
```

# `docker start/stop/restart` 启动/停止/重启

```bash
docker start CONTAINER
docker stop CONTAINER
docker restart CONTAINER
docker stop $(docker ps -q)   # 停止所有运行中的容器
```

# `docker rm` 删除容器

```bash
docker rm CONTAINER
docker rm -f CONTAINER          # 强制删除
docker rm $(docker ps -aq)  # 删除所有已经停止的容器
docker container prune      # 删除所有已停止的容器
```

# `docker exec` 在运行中的容器内执行命令

```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

- `CONTAINER`：容器名或容器 ID。
- `COMMAND`：要在容器内执行的命令。
- `[ARG…]`：传给该命令的参数。

常用选项

- `-i`: 保持标准输入（stdin）打开，允许向容器内输入
- `-t`：分配一个伪终端
- `-d`：后台运行
- `-u`：指定用户
- `-w`：指定工作目录
- `-e`：设置环境变量
- `--privileged`：特权模式

```bash
# 进入 web 容器内部的交互式 shell
docker exec -it web bash

# 进入名为 `mysql` 的容器，并在其中启动 MySQL 客户端，以 root 身份登录。
docker exec -it mysql mysql -uroot -p

# 不进入容器，直接在名为 `web` 的容器里执行 `ls /etc/nginx`，查看 Nginx 配置目录。
docker exec web ls /etc/nginx
```

# `docker cp` 在宿主和容器之间复制文件

```bash
docker cp path1 path2
```

# `docker stats` 查看容器资源使用

```bash
docker stats [OPTIONS] [CONTAINER...]

docker stats
docker stats CONTAINER
docker stats --no-stream
```

常用选项

- `--no-stream`：只输出一次，不持续刷新
- `-a`/`--all`：显示所有容器
- `--format`：自定义输出格式
- `--no-trunc`：不截断输出

# `docker top` 查看容器内进程

```bash
docker top CONTAINER
```

# `docker port` 查看端口映射

```bash
docker port CONTAINER
```

# `docker rename` 重命名容器

```bash
docker rename CONTAINER NEW_NAME
```

# `docker update` 更新容器资源限制

```bash
docker update --memory 1g --cpus 2 CONTAINER
```

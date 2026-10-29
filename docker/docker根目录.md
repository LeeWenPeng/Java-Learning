## 迁移 Docker 存储路径的完整步骤

### 前提

> [!WARNING] 注意：docker 必须建立 Linux 原生文件系统上
>
> **Docker 无法在 NTFS/exFAT 文件系统上运行**。因为 Docker 的核心存储驱动（`overlay2`）需要 Linux 原生文件系统的支持（如写时复制、扩展属性 xattr 等），NTFS 根本不支持这些。

### 1. 检查当前存储位置

先确认一下 Docker 默认的存储路径（通常是 `/var/lib/docker`）：

```bash
docker info -f '{{ .DockerRootDir }}'
```

### 2. 停止 Docker 服务

迁移数据前必须停止 Docker，以防数据损坏：

```bash
sudo systemctl stop docker.socket docker.service
```

### 3. 创建新的存储目录

在你指定的磁盘或分区上创建一个新目录，比如 `/mnt/new_disk/docker`。请确保这个目录所在的分区有足够的空间：

```bash
sudo mkdir -p /path/to/new_disk/docker/
```

### 授权

#### 前提

> 标准的 Docker 守护进程（Daemon）是以 **`root`** 身份运行的。如果你没有特意开启 Rootless Docker，当 Docker 尝试在这个目录下拉取镜像、创建容器或写日志时，它会因为缺乏写入权限而报错（因为 Docker 尝试以 `root` 身份写入一个属于 `leewenpeng` 且没给 `root` 写权限的目录）。
>
> 通常，Docker 数据目录的权限应该是：**所有者为 `root:root`，权限为 `710`（`drwx--x---`）**。

#### 如何操作？

```bash
# 1. 停止 Docker 服务
sudo systemctl stop docker.service

# 2. 修改目录所有者和所属组为 root
sudo chown -R root:root /path/to/new_disk/docker/

# 3. 赋予标准的 Docker 目录权限（可选，但推荐）
sudo chmod 710 /path/to/new_disk/docker/

# 4. 重新启动 Docker
sudo systemctl start docker.service
```

### 4. 迁移现有数据

使用 `rsync` 命令将旧数据完整地**复制**到新位置。推荐使用 `-aP` 参数，它可以保留文件权限并显示进度：

```bash
sudo rsync -aP /var/lib/docker/ /path/to/new_disk/docker/
```

> **注意**：命令中源目录 `/var/lib/docker/` 末尾的斜杠 `/` 非常重要，它表示复制目录**内容**，而不是目录本身。

### 5. 配置 Docker 使用新路径

编辑 Docker 的守护进程配置文件 `/etc/docker/daemon.json`（如果文件不存在就创建它），指定 `data-root` 为新的路径：

```bash
sudo vim /etc/docker/daemon.json
```

在文件中添加以下内容：

```json
{
  "data-root": "/mnt/new_disk/docker"
}
```

> **注意**：如果你的文件里已经有其他配置，请确保 JSON 格式正确，并在最后一项后面加上逗号。

### 6. 重启 Docker 服务并验证

重启 Docker 使配置生效，然后验证新的存储位置是否正确：

```bash
# 启动 Docker
sudo systemctl start docker.service

# 验证存储位置
docker info -f '{{ .DockerRootDir }}'
```

如果命令输出你设置的新路径（例如 `/mnt/new_disk/docker`），就说明迁移成功了。

### 7. 清理旧数据（可选）

在确认一切正常后，可以删除旧的 `/var/lib/docker` 目录来释放磁盘空间。**请务必在确认新环境完全可用后再执行此操作**：

```bash
sudo rm -rf /var/lib/docker
```

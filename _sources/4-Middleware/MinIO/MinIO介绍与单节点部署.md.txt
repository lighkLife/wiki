# MinIO介绍与单节点部署

## 介绍

MinIO 是一款兼容 AWS S3 的存储软件，支持加密、版本管理、删除保留、分布式部署等功能。

MinIO 支持多种安装方式：
- 单节点单硬盘（适用于本地安装，没有任何可靠性，存在单点故障）
- 单节点多硬盘（允许一半的硬盘故障，但性能受限）
- 多节点多硬盘（适用于企业级对象存储，允许一半的节点/硬盘发生故障）

## 单节点单硬盘部署

### 下载安装
```bash
## rpm
wget https://dl.min.io/server/minio/release/linux-amd64/archive/minio-20231207041600.0.0.x86_64.rpm -O minio.rpm
rpm -i install minio.rpm
```
```bash
## deb
wget https://dl.min.io/server/minio/release/linux-amd64/archive/minio_20231207041600.0.0_amd64.deb -O minio.deb
sudo dpkg -i minio.deb
```

### systemd 配置
rpm 或 dpkg 会自动创建 systemd 服务的配置文件，路径为 `/usr/lib/systemd/system/minio.service`，并默认以 minio-user 用户和用户组运行，所以需要创建用户，并赋权：

```bash
groupadd -r minio-user
useradd -M -r -g minio-user minio-user
## /mnt/disk1 /mnt/disk2 /mnt/disk3 是为 MinIO 所挂载的硬盘目录 
chown minio-user:minio-user /mnt/disk1 /mnt/disk2 /mnt/disk3
```

### 创建环境变量
在文件 `/etc/default/minio` 中可添加 MinIO 运行的环境变量。

```ini
# 超管用户名和密码，可以用于登录其自带的 UI 管理系统
MINIO_ROOT_USER=admin
MINIO_ROOT_PASSWORD=password

# 存储目录，代表设置 /data-1,/data-2,/data-3,/data-4 为 minio 的存储卷
# 这里要注意第一次启动时，下面的设置的文件夹必须存在，且为空
MINIO_VOLUMES="/data-{1...4}"

# 服务地址
MINIO_SERVER_URL="http://172.16.11.136:9000"
```

### 启动 minio 服务
```bash
# 设置开机自启动
sudo systemctl enable minio.service
# 启动 MinIO Server
sudo systemctl start minio.service
# 查看运行状态
sudo systemctl status minio.service
```
启动成功后，可以在浏览器中访问上面配置的 `MINIO_SERVER_URL` 进行体验。

## 单节点多硬盘部署

### 注意事项
- 强烈推荐使用 XFS 文件系统来保证性能、可靠性、可预测性和一致性。
- 确保使用相同类型（NVMe, SSD, or HDD）和相同容量的硬盘。

> 即使混合使用不同类型的硬盘，MinIO仍然可以正常工作，只是不会因此获得额外的性能提升。
> 如使用大小不同的硬盘，最大容量受制与容量最小的硬盘。

### 部署
部署步骤与单节点单硬盘相同，区别在设置 `/etc/default/minio` 中的环境变量时，`MINIO_VOLUMES` 设置的时多个硬盘的挂载点。

```ini
# 存储目录，代表设置 /data-1,/data-2,/data-3,/data-4 为 minio 的存储卷
# 这里要注意第一次启动时，下面的设置的文件夹必须存在，且为空
MINIO_VOLUMES="/data-{1...4}"
```
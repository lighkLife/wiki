# MinIO多节点部署

## 环境准备
### 网络和防火墙
开放 minio 服务端口
开放 minio 管理端口
### 连续的主机名
MinIO 不支持分布式部署的非连续主机名或 IP 地址，创建服务器时，MinIO需要使用扩展符号{x...y}来表示一系列连续的 MinIO 主机。如 `minio-0{1...4}.example.com`对应：
```
minio-01.example.com
minio-02.example.com
minio-03.example.com
minio-04.example.com
```
如果主机名或者IP不连续，可以配置`/etc/hosts`来保证主机名连续。
```
# /etc/hosts

198.0.2.10    minio-01.example.net
198.51.100.3  minio-02.example.net
198.0.2.43    minio-03.example.net
198.51.100.12 minio-04.example.net
```
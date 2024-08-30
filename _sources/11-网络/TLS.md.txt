# TLS

## 双向校验

![alt text](img/image.png)

需要提前生成 CA 证书（包括 `ca-pub`， `ca-pri`），再使用 CA 证书生成：
- 服务端证书（公钥在 `server-pub` 中， 私钥在 `server-pri` 中）
- 客户端证书（公钥在 `client-pub` 中， 私钥在 `client-pri` 中）

向服务端提供：
- `ca-pub` (和户端交换公钥证书时，校验客户端证书，实现客户端认证)
- `server-pub` 
- `server-pri`

向客户端提供：
- `ca-pub` (和服务端交换公钥证书时，校验服务端证书，实现服务端认证)
- `client-pub` 
- `client-pri`
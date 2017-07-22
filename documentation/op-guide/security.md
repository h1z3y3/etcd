# 安全模式

etcd 支持自动 TLS 以及通过客户端证书的身份验证, 包括客户端到服务器以及对等（服务器到服务器/集群）的通信。

要开始运行，首先要为成员设置 CA 证书和已签名的密钥对。建议为集群中的每个成员创建并签署一个新的密钥对。

为方便起见，[cfssl]工具为证书生成提供了简单的界面，我们使用[这里] [tls-setup]的工具提供了示例。或者，尝试[生成自签名密钥对的指南] [tls-guide]。

## 基本设置

etcd 通过命令行标志或环境变量来设置几个与证书相关的配置选项:

**客户端到服务器端通讯:**

`--cert-file=<path>`: 用于**到** etcd d的　SSL / TLS连接的证书。当设置此选项时，advertise-client-urls　可以使用　HTTPS　模式。

`--key-file=<path>`: 证书的秘钥, 必须是不加密的.

`--client-cert-auth`: 当这个选项被设置时，etcd 将为受信任CA签名的客户端证书检查所有的传入的 HTTPS 请求，不能提供有效客户端证书的请求将会失败。

`--trusted-ca-file=<path>`: 受信任的认证机构

`--auto-tls`: 为客户端的 TLS 连接，使用自动生成的自签名证书

**对等通讯 (服务器到服务器 / 集群):**

对等选项的工作方式与客户端到服务器的选项相同：

`--peer-cert-file=<path>`: 用于对等体之间的 SSL / TLS 连接的证书。这将用于在对等地址上监听以及向其他对等体发送请求

`--peer-key-file=<path>`: 证书的秘钥, 必须是不加密的.

`--peer-client-cert-auth`: 当这个选项被设置时，etcd 将为受信任CA签名的客户端证书检查所有的传入的对等请求。

`--peer-trusted-ca-file=<path>`: 受信任的认证机构.

`--peer-auto-tls`: 为对等体之间的 TLS 连接使用自动生成的自签名证书

如果提供了客户端到服务器或对等证书，则必须设置密钥。所有这些配置选项也可以通过环境变量 “ETCD_CA_FILE”，“ETCD_PEER_CA_FILE”等提供。

## 示例 1: 用HTTPS的客户端到服务器端传输安全

为此，准备好CA证书（`ca.crt`）和签名密钥对（`server.crt`, `server.key`）。

Let us configure etcd to provide simple HTTPS transport security step by step:

让我们一步一步配置 etcd 来提供简单的 HTTPS 传输安全：

```bash
$ etcd --name infra0 --data-dir infra0 \
  --cert-file=/path/to/server.crt --key-file=/path/to/server.key \
  --advertise-client-urls=https://127.0.0.1:2379 --listen-client-urls=https://127.0.0.1:2379
```

应该可以启动，可以通过使用 HTTPS 访问 etcd 来测试配置：

```bash
$ curl --cacert /path/to/ca.crt https://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar -v
```

该命令应该显示握手成功。 由于我们用自己的证书颁发机构使用自签名证书，CA必须使用`--cacert`选项传递给curl。 另一种可能性是将CA证书添加到系统的可信证书目录（通常位于　`/etc/pki/tls/certs`　或　`/etc/ssl/certs`）中。

**OSX 10.9+ 用户**: curl 7.30.0 在 OSX 10.9+　不能理解通过命令行传递的证书.
取而代之的，将虚拟　ca.crt　直接导入　keychain　或给 curl 添加`-k`　标志来忽略错误。
要测试没有`-k'标志，运行　`open ./fixtures/ca/ca.crt`　并按照提示进行操作。
测试后请删除此证书！
如果有解决方法，请告诉我们。

## 示例 2: 用HTTPS客户端证书的客户端到服务器端认证

现在我们已经给了 etcd 客户端验证服务器身份和提供传输安全性的能力。我们也可以使用客户端证书来防止对 etcd 未经授权的访问。

客户端将向服务器提供证书，服务器将检查证书是否由CA签名，并决定是否服务请求。

为此需要第一个示例中提到的相同文件，以及由同一证书颁发机构签名的客户端(`client.crt`, `client.key`) 密钥对。

```bash
$ etcd --name infra0 --data-dir infra0 \
  --client-cert-auth --trusted-ca-file=/path/to/ca.crt --cert-file=/path/to/server.crt --key-file=/path/to/server.key \
  --advertise-client-urls https://127.0.0.1:2379 --listen-client-urls https://127.0.0.1:2379
```

现在尝试发送与上面相同的请求到服务器：

```bash
$ curl --cacert /path/to/ca.crt https://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar -v
```

该请求会被服务器拒绝：

```bash
...
routines:SSL3_READ_BYTES:sslv3 alert bad certificate
...
```

要想请求成功，我们需要将CA签名的客户端证书发送给服务器：

```bash
$ curl --cacert /path/to/ca.crt --cert /path/to/client.crt --key /path/to/client.key \
  -L https://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar -v
```

输出为：

```bash
...
SSLv3, TLS handshake, CERT verify (15):
...
TLS handshake, Finished (20)
```

还有来自服务器的响应：

```json
{
    "action": "set",
    "node": {
        "createdIndex": 12,
        "key": "/foo",
        "modifiedIndex": 12,
        "value": "bar"
    }
}
```

## 示例 3: 集群中的传输安全和客户端证书

对于**对等通信**，etcd 支持与上述相同的模型，这意味着集群中的 etcd 成员之间的通信。

Assuming we have our `ca.crt` and two members with their own keypairs (`member1.crt` & `member1.key`, `member2.crt` & `member2.key`) signed by this CA, we launch etcd as follows:

假设我们有我们的　`ca.crt` 和两个成员，他们有这个CA签名的自己的 keypairs（`member1.crt`＆`member1.key`，`member2.crt`＆`member2.key`），我们如下启动 etcd：

```bash
DISCOVERY_URL=... # from https://discovery.etcd.io/new

# member1
$ etcd --name infra1 --data-dir infra1 \
  --peer-client-cert-auth --peer-trusted-ca-file=/path/to/ca.crt --peer-cert-file=/path/to/member1.crt --peer-key-file=/path/to/member1.key \
  --initial-advertise-peer-urls=https://10.0.1.10:2380 --listen-peer-urls=https://10.0.1.10:2380 \
  --discovery ${DISCOVERY_URL}

# member2
$ etcd --name infra2 --data-dir infra2 \
  --peer-client-cert-auth --peer-trusted-ca-file=/path/to/ca.crt --peer-cert-file=/path/to/member2.crt --peer-key-file=/path/to/member2.key \
  --initial-advertise-peer-urls=https://10.0.1.11:2380 --listen-peer-urls=https://10.0.1.11:2380 \
  --discovery ${DISCOVERY_URL}
```

etcd 成员将组成一个集群，集群中成员之间的所有通信将使用客户端证书进行加密和验证。etcd的输出将显示其连接的地址使用HTTPS。

## Example 4: 自动自签名安全

对于需要通信加密而需要认证的情况，etcd 支持使用自动生成的自签名证书来加密其消息。这样可以简化部署，因为不需要管理 etcd 以外的证书和密钥。

使用标志`--auto-tls`和`--peer-auto-tls` 配置 etcd 为客户端和对等连接使用自签名证书：

```bash
DISCOVERY_URL=... # from https://discovery.etcd.io/new

# member1
$ etcd --name infra1 --data-dir infra1 \
  --auto-tls --peer-auto-tls \
  --initial-advertise-peer-urls=https://10.0.1.10:2380 --listen-peer-urls=https://10.0.1.10:2380 \
  --discovery ${DISCOVERY_URL}

# member2
$ etcd --name infra2 --data-dir infra2 \
  --auto-tls --peer-auto-tls \
  --initial-advertise-peer-urls=https://10.0.1.11:2380 --listen-peer-urls=https://10.0.1.11:2380 \
  --discovery ${DISCOVERY_URL}
```

自签名证书不会对身份进行验证，因此 crul 将返回错误：

```bash
curl: (60) SSL certificate problem: Invalid certificate chain
```

要禁用证书链检查，请使用　`-k`　标志调用curl：

```bash
$ curl -k https://127.0.0.1:2379/v2/keys/foo -Xput -d value=bar -v
```

## etcd proxy 注意事项

如果连接是安全的，etcd proxy 从其客户端终止TLS，并且使用　`--peer-key-file'　和　'--peer-cert-file'　中指定的代理自己的密钥/证书与 etcd 成员进行通信。

proxy 通过给定成员的 `--advertise-client-urls` 和 `--advertise-peer-urls` 与 etcd 成员进行通信。它将客户端请求转发到 etcd 成员的 advertised client url，并通过 etcd 成员的 advertised peer url 同步初始集群配置。

当 etcd 成员启用客户端身份验证时，管理员必须确保代理的 `--peer-cert-file` 选项中指定的对等证书对于该验证是有效的。如果启用对等身份验证，proxy 的对等证书也必须对对等身份验证有效。

## FAQ

### 使用TLS客户端身份验证时，我看到 SSLv3 警报握手失败？

`golang` 的 `crypto / tls` 包在使用它之前检查证书公钥的 `key usage`。

要使用证书公钥进行客户端认证，我们需要在创建证书公钥时将 `clientAuth` 添加到 `Extended Key Usage`。

这是怎么做的:

1. 将以下部分添加到 openssl.cnf:

    ```bash
    [ ssl_client ]
    ...
      extendedKeyUsage = clientAuth
    ...
    ```

2. 创建证书时，确保在 `-extensions` 标志中引用它:

    ```bash
    $ openssl ca -config openssl.cnf -policy policy_anything -extensions ssl_client -out certs/machine.crt -infiles machine.csr
    ```

### 使用对等证书认证，我收到"证书对127.0.0.1有效，而不是$MY_IP"

确保使用成员的公共IP地址为 Subject 名称来签署证书。例如 `etcd-ca`　工具为它的 `new-cert` 命令提供 `--ip=` 选项。

证书需要在其 Subject 名称中为成员的 FQDN 签名，使用 Subject Alternative Names（短IP SAN）来添加IP地址。`etcd-ca`　工具为它的 `new-cert` 命令提供 `--domain=` 选项，而且　openssl 也可以[这样实现][alt-name]。

[cfssl]: https://github.com/cloudflare/cfssl
[tls-setup]: /hack/tls-setup
[tls-guide]: https://github.com/coreos/docs/blob/master/os/generate-self-signed-certificates.md
[alt-name]: http://wiki.cacert.org/FAQ/subjectAltName




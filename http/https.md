## 为什么需要https
* 隐私。避免传输的数据被窃听。加密。
* 完整性。避免数据被篡改，一般为中间人攻击。
* 身份鉴定。确保通讯对方的身份为合法。证书，数字签名。

## 对称加密 & 非对称加密
* 对称加密。加密和解密过程互逆，加密结果和密钥相关。
* 非对称加密。公钥加密私钥解密，实现`privacy`。私钥加密公钥解密，实现`identification`。

## 握手过程
* `client hello`。客户端发送支持的`SSL/TLS`版本、加密算法列表（`cipher suites`）和随机字符串`client random`。
* `server hello`。服务器根据优先选项，选择版本和算法。发送`SSL/TLS`版本、加密算法、证书（携带公钥）和随机字符串`server random`。
* `client key exchange`。验证证书是否合法，生成`pre-master key`，使用服务器公钥进行加密后发送。
* `change cipher spec`。服务器使用私钥进行解密得到`pre-master key`。\
双方通过`client random`，`server random`和`pre-master key`生成相同的会话密钥。\
客户端发送经过会话密钥加密后的完成消息，服务器回应加密后的完成消息，至此，双方可以使用会话密钥进行对称加密通信。

### cipher suit
1. SSL/TLS
2. 密钥交换算法：RSA，DH，ECDH，ECDHE
3. 证书类型：RSA，DSS
4. 对称加密算法：AES，3DES等
5. 摘要算法：MD5，SHA等

### 其他
* 在一些安全性要求高的场景，服务器可以要求客户端发送证书，以此来验证客户端的身份。
* 其他协商的包括数据压缩算法。

## Diffie-Hellman握手过程
* `client hello`。客户端发送支持的`SSL/TLS`版本、加密算法列表（`cipher suites`）和随机字符串`client random`。
* `server hello`。服务器根据优先选项，选择版本和算法。发送`SSL/TLS`版本、加密算法、证书（携带公钥）和随机字符串`server random`。并且包含数字签名后的`client random`、`server random`、`DH parameter`。
* `client DH parameter`。验证证书是否合法，数字签名是否正确，发送`DH parameter`。
* 双方使用`DH parameter`生成`pre-master key`，之后步骤和上面一样。

### Diffie-Hellman key exchange
* `g(generator)`，`p(big prime number)`。在开始前预知的，一般在标准中写明。
* 客户端随机生成`a(1 ~ p)`，计算`g^a % p`。
* 服务器随机生成`b(1 ~ p)`，计算`g^b % p`。
* 客户端接收到`g^b % p`，计算`(g^b)^a % p`，即`g^ab % p`。
* 服务器接收到`g^a % p`，计算`(g^a)^b % p`，即`g^ab % p`。
* 最终双方得到相同的密钥。

### 使用DH密钥交换算法原因
具备更好的向前安全性，因为双方的私有变量会在计算完`pre-master key`后丢弃，降低泄密的风险。

## 证书
* `Root Store`：存储受信任的`CA`的数据库。
* `Certificate Authority`：证书机构，确认证书拥有者身份，发行证书，验证证书的有效性。
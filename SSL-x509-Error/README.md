## 问题描述
使用自签名的ssl证书遇到如下问题：
tls: failed to verify certificate: x509: certificate relies on legacy Common Name field, use SANs instead
### 问题原因
出现错误 tls: failed to verify certificate: x509: certificate relies on legacy Common Name field, use SANs instead 的原因是在创建 SSL/TLS 证书时，证书依赖于传统的 Common Name (CN) 字段，而没有使用现代标准所推荐的 Subject Alternative Names (SANs) 字段。现代的 TLS 客户端（比如最新版本的浏览器和安全工具）要求证书使用 SANs 字段来指定有效的主机名。

要解决这个问题，需要生成一个新的自签名证书，确保在证书中包含 SANs。
相关问题可以参考 [How do I use SANs with openSSL instead of common name](https://stackoverflow.com/questions/64814173/how-do-i-use-sans-with-openssl-instead-of-common-name)
### 解决思路
问题的关键是要在生成SSL证书的时候添加：-addext "subjectAltName = DNS:<你的域名>"，具体操作如下：

```bash
openssl req -newkey rsa:4096 -nodes -sha256 -keyout domain.key -x509 -days 36500 -out domain.crt -addext "subjectAltName = DNS:<你的域名>"
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:
State or Province Name (full name) []:
Locality Name (eg, city) [Default City]:
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:<你的域名>
Email Address []:
```
### 遇到"unknown option -addext"
遇到 unknown option -addext 错误时，这通常意味着你正在使用的 OpenSSL 版本不支持 -addext 选项。-addext 选项是在较新版本的 OpenSSL 中引入的，用于直接在命令行中添加扩展到证书请求中。如果你的 OpenSSL 版本不支持这个选项，你可以通过创建一个配置文件来添加主题备用名称（Subject Alternative Name, SAN）。

相关问题可以参考Harbor仓库官方提供的生成SSL证书方式 [Harbor仓库SSL证书生成方式](https://goharbor.io/docs/2.9.0/install-config/configure-https/)

关键点是要生成"v3.ext"文件，可以看到其中有关于"alt_names"的设置。并且在生成证书的时候通过"-extfile"应用该文件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/e3a51ba5b1954e7181de250f386fc093.png)

# 使用OpenSSL创建根证书并签发子证书

> 总是有一些场景需要自行签发一系列证书并且让系统信任。如果每次都签发一个新的证书，显然会比较麻烦。所以只需要一个根证书，让系统/浏览器信任这个证书，后续这个根证书签发的证书都是可以被信任的了，非常方便。


## 整体步骤

- 创建根证书

## 创建根证书

创建基础配置文件`openssl-ca.cnf`并写入基本配置

```shell
touch openssl-ca.cnf
```

写入配置

```properties
HOME            = .
RANDFILE        = $ENV::HOME/.rnd

####################################################################
[ ca ]
default_ca    = CA_default      # The default ca section

[ CA_default ]

default_days     = 365         # How long to certify for
default_crl_days = 30           # How long before next CRL
default_md       = sha256       # Use public key default MD
preserve         = no           # Keep passed DN ordering

x509_extensions = ca_extensions # The extensions to add to the cert

email_in_dn     = no            # Don't concat the email in the DN
copy_extensions = copy          # Required to copy SANs from CSR to cert

####################################################################
[ req ]
default_bits       = 4096
default_keyfile    = cakey.pem
distinguished_name = ca_distinguished_name
x509_extensions    = ca_extensions
string_mask        = utf8only

####################################################################
[ ca_distinguished_name ]
countryName         = Country Name (2 letter code)
countryName_default = CN

stateOrProvinceName         = State or Province Name (full name)
stateOrProvinceName_default = Province

localityName                = Locality Name (eg, city)
localityName_default        = City

organizationName            = Organization Name (eg, company)
organizationName_default    = Test CA, Limited

organizationalUnitName         = Organizational Unit (eg, division)
organizationalUnitName_default = Server Research Department

commonName         = Common Name (e.g. server FQDN or YOUR name)
commonName_default = Test CA

emailAddress         = Email Address
emailAddress_default = test@example.com

####################################################################
[ ca_extensions ]

subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always, issuer
basicConstraints       = critical, CA:true
keyUsage               = keyCertSign, cRLSign
```

以上内容是签发一个CA证书必需的配置内容，从`openssl.cnf`文件中提取出来，方便操作。

开始签发证书

```shell
$ openssl req -x509 -config openssl-ca.cnf -newkey rsa:4096 -sha256 -nodes -out cacert.pem -outform PEM
```

这里`-nodes`参数表示不使用des进行加密，可以去掉这个参数，就可以使用密码保护证书了。

## 创建子证书

创建子证书配置文件`openssl-server.cnf`并写入配置文件

```shell
touch openssl-server.cnf
```

写入配置文件

```prop
HOME            = .
RANDFILE        = $ENV::HOME/.rnd

####################################################################
[ req ]
default_bits       = 2048
default_keyfile    = serverkey.pem
distinguished_name = server_distinguished_name
req_extensions     = server_req_extensions
string_mask        = utf8only

####################################################################
[ server_distinguished_name ]
countryName         = Country Name (2 letter code)
countryName_default = CN

stateOrProvinceName         = State or Province Name (full name)
stateOrProvinceName_default = Province

localityName         = Locality Name (eg, city)
localityName_default = City

organizationName            = Organization Name (eg, company)
organizationName_default    = Test Server, Limited

commonName           = Common Name (e.g. server FQDN or YOUR name)
commonName_default   = Test Server

emailAddress         = Email Address
emailAddress_default = test@example.com

####################################################################
[ server_req_extensions ]

subjectKeyIdentifier = hash
basicConstraints     = CA:FALSE
keyUsage             = digitalSignature, keyEncipherment
subjectAltName       = @alternate_names
nsComment            = "OpenSSL Generated Certificate"

####################################################################
[ alternate_names ]

DNS.1  = example.com
DNS.2  = www.example.com
DNS.3  = mail.example.com
DNS.4  = ftp.example.com
```

如果没有域名，在最后的`[alternate_names]`部分需要明确的标注你当前证书需要部署在的服务器的IP地址。如果部署在本地则应该为：

```properties
IP.1 = 127.0.0.1
IP.2 = ::1
IP.3 = 192.168.0.3
```

然后，使用下面的命令创建服务器子证书

```shell
openssl req -config openssl-server.cnf -newkey rsa:2048 -sha256 -nodes -out servercert.csr -outform PEM
```

这个命令会在当前目录生成两个文件，`servercert.csr`表示给公共根证书认证机构签发认证内容的公共信息，`serverkey.pem`就是生成的私钥。同样，可以不使用`-nodes`来给这个证书加密，但是绝大多数数服务器（例如`nginx`）不支持配置加密的证书。

## 签发证书

对刚刚的`openssl-ca.cnf`配置文件追加签发证书的配置信息。

在最后面，增加下面内容：

```proper
####################################################################
[ signing_policy ]
countryName            = optional
stateOrProvinceName    = optional
localityName           = optional
organizationName       = optional
organizationalUnitName = optional
commonName             = supplied
emailAddress           = optional

####################################################################
[ signing_req ]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
basicConstraints       = CA:FALSE
keyUsage               = digitalSignature, keyEncipherment
```

然后，在`[CA_default]`节中增加以下内容

```properties
base_dir      = .
certificate   = $base_dir/cacert.pem   # The CA certifcate
private_key   = $base_dir/cakey.pem    # The CA private key
new_certs_dir = $base_dir              # Location for new certs after signing
database      = $base_dir/index.txt    # Database index file
serial        = $base_dir/serial.txt   # The current serial number

unique_subject = no  # Set to 'no' to allow creation of
                     # several certificates with same subject.
```

创建数据库索引文件和序列号文件，这两个文件分别配置在上述的`database`节和`serial`节中

```shell
touch index.txt
echo '01' > serial.txt
```

执行签发命令

```shel
openssl ca -config openssl-ca.cnf -policy signing_policy -extensions signing_req -out servercert.pem -infiles servercert.csr
```

然后会显示一系列需要签发的内容信息，并让你确认是否对证书进行签发，输入`y`表示确认签发；之后，会再次确认，再次输入`y`即可。

此时，你就得到了签发完成的证书文件`servercert.pem`和私钥文件`serverkey.pem`。

## 附

1. 
验证证书内容，可以使用如下命令查看证书内容。
```shell
openssl x509 -in 证书 -text -noout
```


2. 
验证申请内容，可以使用如下命令。
```shell
openssl req -text -noout -verify -in XXX.csr
```


3. 
如果需要将证书配置到`nginx`中，nginx默认需要的是crt格式的证书，可以使用命令将生成的pem证书转为crt格式。
```shell
openssl x509 -text -in servercert.pem -out servercert.crt
```


4. 
同3，可以将`cacert.pem`转为`cacert.crt`以便能够在windows下双击导入，注意导入的时候，需要将证书安装到“受信任的根证书颁发机构”存储目录中。

5. 
在Mac环境下可以使用`keychain Access.app`管理自定义私钥，也可以使用命令导入
```shell
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain /path/to/cert/cacert.crt
```



## 参考内容

[[How do you sign a Certificate Signing Request with your Certification Authority?](https://stackoverflow.com/questions/21297139/how-do-you-sign-a-certificate-signing-request-with-your-certification-authority)]([https://stackoverflow.com/questions/21297139/how-do-you-sign-a-certificate-signing-request-with-your-certification-authority/21340898](https://stackoverflow.com/questions/21297139/how-do-you-sign-a-certificate-signing-request-with-your-certification-authority/21340898))

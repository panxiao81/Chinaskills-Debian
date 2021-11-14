# 证书认证与 OpenSSL

> 参考资料：
>
> [如何使用 OpenSSL 建立開發測試用途的自簽憑證 (Self-Signed Certificate)](https://blog.miniasp.com/post/2019/02/25/Creating-Self-signed-Certificate-using-OpenSSL)
>
> [Create a self-signed certificate using OpenSSL](https://blog.cssuen.tw/create-a-self-signed-certificate-using-openssl-240c7b0579d3)
> 
> [OpenSSL Certification Authority (CA) on Ubuntu Server](https://networklessons.com/uncategorized/openssl-certification-authority-ca-ubuntu-server)
> 
> [Create your own Certificate Authority](https://priyalwalpita.medium.com/create-your-own-certificate-authority-47f49d0ba086)

现在生产环境最次其实也应该有个 Let‘s Encrypt，而且能够被浏览器信任，自签证书除了测试环境其实没啥用（ 

## 实验

本例我们需要自己扮演一个根证书颁发机构 ( CA )，来自己给自己签发一个根证书，并用这个根证书为其他证书签名。

这个实验需要使用 OpenSSL 来签发证书。

这个实验需要几个步骤，分别为:

- 生成根证书所需的私钥
- 用这个私钥签发证书，作为根证书
- 生成需要证书的客户端的私钥
- 使用客户端私钥创建证书签发请求
- 在根证书服务器上签发证书

在生产环境中，要签发实际使用的证书通常在我们的本机上生成私钥，来确保私钥的安全性，但实验中，我们完全可以把所有环节均在 CA 服务器上完成，这样可以省下许多把文件传来传去的时间。

### 根证书的签发

签发根证书前，先做一些配置。

修改 `/usr/lib/ssl/openssl.cnf`，在 48 行，`[CA_default]` 中，修改 `dir` 为存放 CA 需要文件的根目录，本例为 `/CA`

修改后，创建两个文件

```console
$ # 用于定位根目录位置
$ touch /CA/index.txt
$ # 一个初始数字用于代表签发证书的数量，每签发一张证书，数字便会 +1
$ echo 1000 > /CA/serial
```

创建好目录结构

```console
$ cd /CA
$ mkdir newcerts private certs
```

接下来切换到 CA 的根目录，开始生成 CA 根证书

```console
$ cd /CA
$ # 根证书的私钥，即 key 文件，长度为 4096 bit
$ openssl genrsa -out cakey.pem 4096
$ mv cakey.pem ./private/
$ # 用刚才生成的私钥签发证书
$ openssl req -new -x509 -key ./private/cakey.pem -out ./cacert.pem
$ # 根据需求输入证书信息
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Liaoning
Locality Name (eg, city) []:Dandong
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Inc
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:Skill Global Root CA
Email Address []:
```

生成的 `cacert.pem` 即为根证书，需要将其导入所有主机的信任列表

### 其他证书的签发

签发其他证书时，依然需要生成私钥，但第二步需要生成证书签发请求，然后使用 CA 根证书签名

```console
$ # 生成一个私钥
$ openssl genrsa -out server01key.pem 4096
$ # 使用私钥生成一个证书签发请求
$ openssl req -new -key server01key.pem -out server01.csr
ou are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Liaoning
Locality Name (eg, city) []:Dandong
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Inc
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:server01.sdskills.com
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
$ # 用根证书与私钥签发证书
$ openssl ca -in server01.csr -out server01.pem
Using configuration from /usr/lib/ssl/openssl.cnf
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 4096 (0x1000)
        Validity
            Not Before: Mar 28 14:36:39 2021 GMT
            Not After : Mar 28 14:36:39 2022 GMT
        Subject:
            countryName               = CN
            stateOrProvinceName       = Liaoning
            organizationName          = Inc
            commonName                = server01.sdskills.com
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                29:55:53:D7:48:A6:93:6A:56:9C:E5:38:C3:92:6A:2F:3C:AC:B7:06
            X509v3 Authority Key Identifier: 
                keyid:18:D9:48:2B:0E:D2:F4:FF:0C:0D:67:E8:CB:26:4C:30:CE:F2:6D:60

Certificate is to be certified until Mar 28 14:36:39 2022 GMT (365 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
```

连敲两个 `y` 即可

生成的 server01.pem 即为证书

在配置证书时，需要生成的客户机自己的私钥，与经过根证书签名的证书，根证书自己的私钥需在 CA 服务器上自己保存好。

之后要签发证书，按照生成私钥 -> 生成签发请求 -> 在 CA 上签名 这三步即可。

由于总是要传一些签发好的证书，所以建议这时先不要阻挡 CA 服务器的 SSH 访问，能大大加快实际做题的速度

> 指用 SCP 传文件

证书需要导入进其他主机的根证书信任列表，我个人建议用 Ansible，反正 APT 源里有

将 CA 根证书复制到主机的 `/usr/local/share/ca-certificates/` 文件夹，之后运行 `update-ca-certificates`

Ansible Playbook 如下：

```yml
---
- hosts: all
  vars: 
    ca: 
      - cacert.pem
  tasks:
    - name: copy ca file
      copy:
        src: '{{ item }}'
        dest: /usr/local/share/ca-certificates
        mode: 0644
        owner: root
        group: root
      loop: '{{ ca }}'
      notify: update trusted-ca
  handlers:
    - name: update trusted-ca
      shell: /usr/sbin/update-ca-certificates
```

另外还需要导入进 Firefox 的证书信任列表，这样就不会在访问时弹出安全警告了。
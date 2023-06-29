## 安装 openssl
在 ubuntu 系统下，安装命令如下：
```
sudo apt install -y openssl libssl-dev
```

在 centos 系统下，安装命令如下：
```
yum install -y openssl openssl-devel
```

## skynet 编译开启 tls 模块
修改 Makefile 文件，找到 `#TLS_MODULE=ltls` 这一行，删除该行前的的注释符 `#`。
```
TLS_MODULE=ltls
TLS_LIB=
TLS_INC=
```
TLS_LIB 和 TLS_INC 如果用系统的包管理命令安装的可以不指定，如果自己编译安装的需要自己指定 lib 和 inclue 的路径。
修改好后，再次编译 skynet。也可以单独对 tls 进行编译，命令如下：
```
make luaclib/ltls.so -B
```

## 在 Lua 中启用 ssl
若要在 lua 服务中使用 ssl，则要在配置文件中启用 `enablessl = true`。

## 自签名证书
在内网开发环境下，使用都是 ip 或者 内网测试域名，为了测试 ssl，一般都是使用自签名证书。自签名证书生成如下：
```
# 注意：不能在fish下使用，因为fish 不支持 <(xxx) 的语法
openssl req \
-newkey rsa:2048 \
-x509 \
-nodes \
-keyout file.key \
-new \
-out file.crt \
-subj /C=CN/ST=GuangDong/L=GuangZhou/CN=xxx.com \
-reqexts SAN \
-extensions SAN \
-config <(cat /etc/ssl/openssl.cnf \
    <(printf '[SAN]\nsubjectAltName=DNS.1:www.xxx.com,DNS.2:*.xxx.com,IP.1:192.168.1.101')) \
-sha256 \
-days 3650
```

为了让 ip 地址也能绑定到证书，需要设置 `SubjectAltName` 扩展项，参考[证书签发与 SubjectAltName 扩展项](https://zhuanlan.zhihu.com/p/157911310)。

关于 openssl 如何生成证书以及参数的意义，可以参考[使用OpenSSL生成多域名自签名证书](https://blog.csdn.net/fansys/article/details/115615564)。

生成的自签名证书需要导入到系统才能使用，在 windows 系统下导入步骤如下：
1. 按下 win+r 组合键，输入 `certmgr.msc`，右键选中【受信任的根证书颁发机构】->【所有任务】->【导入】
2. 安装引导步骤，选择生成的证书（注意不要选到私钥文件了），一路点击确认即可导入。

## openssl库兼容问题
不同版本的 linux 系统安装的 openssl 的版本也不一样，例如 Ubuntu22.04 安装的是 3.0.2 版本的 openssl，Centos8.5 安装的是 1.1.1 版本。

查看 openssl 版本命令如下：
```
openssl version
或者
openssl version -a
```

若想查看动态库所对应的版本，可以通过以下方式：
```
strings libssl.so |grep -i "^openssl"
strings libcrypto.so |grep -i "^openssl"
```
在输出内容中可以找到版本号，例如：`OpenSSL 3.0.2 15 Mar 2022`。

另外，在 Centos 上，若出现`libcrypto.so.10 no found`或`libssl.so.10 no found`，说明程序链接的是 1.0 版本的 openssl 库，可以`yum install compat-openssl10 -y` 来解决。
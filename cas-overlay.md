# cas-overlay服务端部署

### 基于静态用户配置部署

下载

[cas-overlay-template-5.2.zip](https://github.com/apereo/cas-overlay-template/tree/5.2)

进入到解压开的目录，执行build run

启动失败，报错：

Caused by: java.io.FileNotFoundException: **\etc\cas\thekeystore** (系统找不到指定的
文件。)

运行之前需要先生成keystore文件

```
https://github.com/apereo/cas-overlay-template/tree/5.2

Deployment
Create a keystore file thekeystore under /etc/cas. Use the password changeit for both the keystore and the key/certificate entries.
Ensure the keystore is loaded up with keys and certificates of the server.
```

这里需要创建keystorefile, 参考2: [CAS 5.1.x 的搭建和使用（一）—— 通过Overlay搭建服务端](https://www.cnblogs.com/flying607/p/7598248.html)

C:\Users\gh>keytool -genkey -alias cas -keyalg RSA -keysize 2048 -keypass 123456
 -storepass 123456 -keystore D:/etc/cas/cas.keystore -dname "CN=cas.example.org,
OU=cas.com,O=cas,L=XM,ST=XM,C=CN"

Warning:
JKS 密钥库使用专用格式。建议使用 "keytool -importkeystore -srckeystore D:/etc/ca
s/cas.keystore -destkeystore D:/etc/cas/cas.keystore -deststoretype pkcs12" 迁移
到行业标准格式 PKCS12。

hosts文件加上

```
127.0.0.1 cas.example.org
```

新建src/main/resources目录，设置为资源目录类型

将overlays/org.apereo.cas.cas-server-webapp-tomcat-5.2.6/WEB-INF/classes/application.properties

拷贝到resources目录之下，修改以下配置：

```
server.ssl.key-store=file:/etc/cas/cas.keystore
server.ssl.key-store-password=123456
server.ssl.key-password=123456
```

再执行build run启动，就启动成功了

访问:  https://cas.example.org:8443/cas/

输入casuser/Meloon登录成功





## 参考

1. [WAR Overlay Installation](https://apereo.github.io/cas/5.2.x/installation/Maven-Overlay-Installation.html)

2. [CAS 5.1.x 的搭建和使用（一）—— 通过Overlay搭建服务端](https://www.cnblogs.com/flying607/p/7598248.html)




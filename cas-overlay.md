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



### 基于mysql验证用户

建表

```sql
CREATE TABLE `auth_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '自增主键',
  `name` varchar(255) NOT NULL COMMENT '姓名',
  `account` varchar(255) NOT NULL COMMENT '账号',
  `passwd` varchar(255) NOT NULL COMMENT '密码',
  `mobile` varchar(255) NOT NULL COMMENT '手机',
  `email` varchar(255) NOT NULL COMMENT '邮箱',
  `type` int(255) NOT NULL DEFAULT '0' COMMENT '类型 0-普通用户',
  `status` int(11) NOT NULL DEFAULT '0' COMMENT '状态 0-有效 1-失效',
  `remark` varchar(255) NOT NULL COMMENT '备注',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `u_account` (`account`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;


```

插入测试用户数据

```sql
INSERT INTO `auth_user` (`id`, `name`, `account`, `passwd`, `mobile`, `email`, `type`, `status`, `remark`, `create_time`, `update_time`) VALUES ('1', 'admin', 'admin', 'admin123', '13306011713', '1', '0', '0', '1', '2018-12-06 14:21:56', '2018-12-06 14:21:56');

```



https://apereo.github.io/cas/5.2.x/installation/JDBC-Drivers.html

引入依赖

```

    <dependencies>
        <dependency>
            <groupId>org.apereo.cas</groupId>
            <artifactId>cas-server-webapp${app.server}</artifactId>
            <version>${cas.version}</version>
            <type>war</type>
            <scope>runtime</scope>
        </dependency>

        <!-- 引入数据库认证相关 start-->
        <dependency>
            <groupId>org.apereo.cas</groupId>
            <artifactId>cas-server-support-jdbc</artifactId>
            <version>${cas.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apereo.cas</groupId>
            <artifactId>cas-server-support-jdbc-drivers</artifactId>
            <version>${cas.version}</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.36</version>
        </dependency>
        <!--数据库认证相关 end-->

    </dependencies>
```



resources/application.properties下

注释掉默认配置

```
# cas.authn.accept.users=casuser::Mellon
```

新增jdbc的配置

https://apereo.github.io/cas/5.2.x/installation/Configuration-Properties.html#database-authentication

```
cas.authn.jdbc.query[0].sql=SELECT * FROM auth_user WHERE account=?
cas.authn.jdbc.query[0].healthQuery=
cas.authn.jdbc.query[0].isolateInternalQueries=false
cas.authn.jdbc.query[0].url=jdbc:mysql://127.0.0.1:3306/shadow?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&useSSL=false
cas.authn.jdbc.query[0].failFastTimeout=1
cas.authn.jdbc.query[0].isolationLevelName=ISOLATION_READ_COMMITTED
cas.authn.jdbc.query[0].dialect=org.hibernate.dialect.HSQLDialect
cas.authn.jdbc.query[0].leakThreshold=10
cas.authn.jdbc.query[0].propagationBehaviorName=PROPAGATION_REQUIRED
cas.authn.jdbc.query[0].batchSize=1
cas.authn.jdbc.query[0].user=root
# cas.authn.jdbc.query[0].ddlAuto=create-drop
cas.authn.jdbc.query[0].maxAgeDays=180
cas.authn.jdbc.query[0].password=root
cas.authn.jdbc.query[0].autocommit=false
cas.authn.jdbc.query[0].driverClass=com.mysql.jdbc.Driver
cas.authn.jdbc.query[0].idleTimeout=5000
# cas.authn.jdbc.query[0].credentialCriteria=
# cas.authn.jdbc.query[0].name=
# cas.authn.jdbc.query[0].order=0
# cas.authn.jdbc.query[0].dataSourceName=
# cas.authn.jdbc.query[0].dataSourceProxy=false
# Hibernate-specific properties (i.e. `hibernate.globally_quoted_identifiers`)
# cas.authn.jdbc.query[0].properties.propertyName=propertyValue

cas.authn.jdbc.query[0].fieldPassword=passwd
# cas.authn.jdbc.query[0].fieldExpired=
# cas.authn.jdbc.query[0].fieldDisabled=
# cas.authn.jdbc.query[0].principalAttributeList=sn,cn:commonName,givenName

# cas.authn.jdbc.query[0].passwordEncoder.type=NONE|DEFAULT|STANDARD|BCRYPT|SCRYPT|PBKDF2|com.example.CustomPasswordEncoder
# cas.authn.jdbc.query[0].passwordEncoder.characterEncoding=
# cas.authn.jdbc.query[0].passwordEncoder.encodingAlgorithm=
# cas.authn.jdbc.query[0].passwordEncoder.secret=
# cas.authn.jdbc.query[0].passwordEncoder.strength=16

# cas.authn.jdbc.query[0].principalTransformation.pattern=(.+)@example.org
# cas.authn.jdbc.query[0].principalTransformation.groovy.location=file:///etc/cas/config/principal.groovy
# cas.authn.jdbc.query[0].principalTransformation.suffix=
# cas.authn.jdbc.query[0].principalTransformation.caseConversion=NONE|UPPERCASE|LOWERCASE
# cas.authn.jdbc.query[0].principalTransformation.prefix=
```

重新执行build run

用数据库中的用户名密码admin/admin123登录成功



## 参考

1. [WAR Overlay Installation](https://apereo.github.io/cas/5.2.x/installation/Maven-Overlay-Installation.html)
2. [CAS 5.1.x 的搭建和使用（一）—— 通过Overlay搭建服务端](https://www.cnblogs.com/flying607/p/7598248.html)
3. [Database Authentication](https://apereo.github.io/cas/5.2.x/installation/Database-Authentication.html)
4. [Configuration-Properties.html#database-authentication](https://apereo.github.io/cas/5.2.x/installation/Configuration-Properties.html#database-authentication)






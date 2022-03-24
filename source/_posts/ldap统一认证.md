title: ldap统一认证
author: Wtli
date: 2022-01-07 08:59:10
tags:
---
认证服务相关框架学习：

- ldap：轻型目录访问协议
- NIS：网络信息服务
- OpenLdap
- ad
- radius
- AAA协议

<!--more-->

#### LDAP

ldap是什么？

定义：

轻型目录访问协议（英文：Lightweight Directory Access Protocol，缩写：LDAP，/ˈɛldæp/）是一个开放的，中立的，工业标准的应用**协议**，通过IP协议提供访问控制和维护分布式信息的目录信息。

为什么使用ldap？

ldap目录与普通数据库的主要不同之处在于数据的组织方式，它是一种有层次的、**树形**结构。  
ldap使用一种树形的数据结构，这就说明在查找方面比普通的数据库效率更高。

ldap主要应用场景？

1. 网络服务：DNS服务
2. 统一认证服务：
3. Linux PAM (ssh, login, cvs. . . )
4. Apache访问控制
5. 各种服务登录(ftpd, php based, perl based, python based. . . )
6. 个人信息类，如地址簿
7. 服务器信息，如帐号管理、邮件服务等

ldap相关产品？

主要的LDAP 服务器
1. OpenLDAP: http://www.openldap.org/
- Microsoft Active Directory
- Sun one Directory
- IBM directory server
- Netscape LDAP SDK


#### NIS

NIS是Sun Microsystem于1985年发布的一项目录服务技术（Diretory Service），用来集中控制几个系统管理数据库的网络用品。简化了UNIX和LINUX桌面客户的管理工作，客户端利用它可以使用中心服务器的管理文件。

LDAP与NIS相比

1. LDAP是标准的、跨平台的，在Windows下也能支持。
2. LDAP支持非匿名的访问，而且有比较复杂的访问控制机制(如ACL)，安全性似乎更好一些。
3. LDAP支持很多复杂的查询方式。
4. LDAP的用途较NIS更为广泛，各种服务都可以和LDAP挂钩。


#### OpenLdap

OpenLDAP项目是一个共同开发的项目，旨在开发一个健壮的、商业级的、功能齐全的、开放源码的LDAP应用程序和开发工具套件。该项目由世界范围内的志愿者社区管理，他们使用Internet进行通信、计划和开发OpenLDAP套件及其相关文档。


##### OpenLdap部署

**下载安装包**

按照OpenLdap软件[下载页面](http://www.openldap.org/software/download/)的说明获取该软件的副本。建议新用户从最新版本开始。

**解压缩**

![upload successful](/images/pasted-105.png)

**查看文档**

可跳过。<font color='gray'>查看随发行版提供的版权、许可、README和安装文档。版权和许可协议提供了有关OpenLDAP软件可接受的使用、复制和保修限制的信息。  您还应该阅读本文档的其他章节。特别是，本文档的构建和安装OpenLDAP软件一章提供了关于必备软件和安装过程的详细信息。</font>

**运行**

运行提供的配置脚本来配置在您的系统上构建的发行版。configure脚本接受许多命令行选项，可以启用或禁用可选的软件特性。通常，默认值是可以的，但您可能需要更改它们。要获取配置接受的完整选项列表，请使用——help选项:

```
$ ./configure --help
```

![upload successful](/images/pasted-106.png)

当然，也可以默认安装直接运行

```
$ ./configure
```
成功运行

![upload successful](/images/pasted-107.png)

**编译软件**

下一步是构建软件。这一步有两部分，首先我们构建依赖，然后编译软件:

```
$ make depend
$ make
```

虽然log显示的和报错一样，但是没有error的字样，就当作没报错。。

![upload successful](/images/pasted-108.png)

**测试编译**

感觉同样可跳过。

为了确保正确的构建，你应该运行测试套件(只需要几分钟):
```
$ make test
```

**安装软件**

现在，您已经准备好安装软件;这通常需要超级用户权限:

```
$ su root -c 'make install'
```

现在一切都应该安装在/usr/local下(或者configure使用的任何安装前缀)。

注：在mac下要使用
```
$ sudo su root -c 'make install'
```

**配置**

使用你最喜欢的编辑器编辑提供的slapd.ldif(通常安装在/usr/local/etc/openldap/slapd.ldif)，包含一个MDB数据库定义的形式:

实例：
```
dn: olcDatabase=mdb,cn=config
objectClass: olcDatabaseConfig
objectClass: olcMdbConfig
olcDatabase: mdb
OlcDbMaxSize: 1073741824
olcSuffix: dc=<MY-DOMAIN>,dc=<COM>
olcRootDN: cn=Manager,dc=<MY-DOMAIN>,dc=<COM>
olcRootPW: secret
olcDbDirectory: /usr/local/var/openldap-data
olcDbIndex: objectClass eq
```

请确保将\<MY-DOMAIN\>和\<COM\>替换为域名的适当域组件。例如，For example.com，使用:

```
dn: olcDatabase=mdb,cn=config
objectClass: olcDatabaseConfig
objectClass: olcMdbConfig
olcDatabase: mdb
OlcDbMaxSize: 1073741824
olcSuffix: dc=example,dc=com
olcRootDN: cn=Manager,dc=example,dc=com
olcRootPW: secret
olcDbDirectory: /usr/local/var/openldap-data
olcDbIndex: objectClass eq
```

如果您的域包含其他组件，如eng.uni.edu.eu，使用:

```
dn: olcDatabase=mdb,cn=config
objectClass: olcDatabaseConfig
objectClass: olcMdbConfig
olcDatabase: mdb
OlcDbMaxSize: 1073741824
olcSuffix: dc=eng,dc=uni,dc=edu,dc=eu
olcRootDN: cn=Manager,dc=eng,dc=uni,dc=edu,dc=eu
olcRootPW: secret
olcDbDirectory: /usr/local/var/openldap-data
olcDbIndex: objectClass eq
```

**导入数据**

通过运行以下命令来导入配置数据库:
```
$ su root -c /usr/local/sbin/slapadd -n 0 -F /usr/local/etc/slapd.d -l /usr/local/etc/openldap/slapd.ldif
```

**启动SLDAP**

启动独立的LDAP守护进程:
```
$ su root -c /usr/local/libexec/slapd -F /usr/local/etc/slapd.d
```

下面是运行参数，使用命令-F来挂载配置目录
```
sh-3.2# /usr/libexec/slapd --help
/usr/libexec/slapd: illegal option -- -
usage: /usr/libexec/slapd options
	-4		IPv4 only
	-6		IPv6 only
	-T {acl|add|auth|cat|dn|index|passwd|test}
			Run in Tool mode
	-c cookie	Sync cookie of consumer
	-d level	Debug level
	-f filename	Configuration file
	-F dir	Configuration directory
	-g group	Group (id or name) to run as
	-h URLs		List of URLs to serve
	-l facility	Syslog facility (default: LOCAL4)
	-n serverName	Service name
	-o <opt>[=val] generic means to specify options; supported options:
		slp[={on|off|(attrs)}] enable/disable SLP using (attrs)
	-r directory	Sandbox directory to chroot to
	-s level	Syslog level
	-u user		User (id or name) to run as
	-V		print version info (-VV exit afterwards, -VVV print
			info about static overlays and backends)
```

要检查服务器是否正在运行并配置正确，可以使用ldapsearch(1)对其运行搜索。默认情况下，ldapsearch的安装路径为/usr/local/bin/ldapsearch:

```
$ ldapsearch -x -b '' -s base '(objectclass=*)' namingContexts
```

注意，命令参数周围使用单引号，以防止shell解释特殊字符。这应该返回:

```
dn:
namingContexts: dc=example,dc=com
```

**添加数据**

使用ldapadd向LDAP目录添加条目。ldapadd期望以LDIF形式输入。我们将分两个步骤:
- 创建LDIF文件 
- 运行用ldapadd

创建一个LDIF文件，包含:
```
dn: dc=example,dc=com
objectclass: dcObject
objectclass: organization
o: Example Company
dc: example

dn: cn=Manager,dc=example,dc=com
objectclass: organizationalRole
cn: Manager
```
运行ldapadd(1)将这些条目插入到目录中。

```
$ ldapadd -x -D "cn=Manager,dc=example,dc=com" -W -f example.ldif
```

**查询插入数据**

验证添加的条目是否在您的目录中。可以使用任何LDAP client来完成此操作，下面使用了ldapsearch工具。

```
$ ldapsearch -x -b 'dc=example,dc=com' '(objectclass=*)'
```







#### OpenLdap配置说明

##### cn=config

这个条目中包含的指令通常作为一个整体应用于服务器。它们大多数是面向系统或连接的，与数据库无关。这个条目必须有objectClass: olcGlobal。

**olcIdleTimeout: \<integer\>**

指定强制关闭空闲客户端连接前需要等待的秒数。默认值为0，表示禁用该特性。

**olcLogLevel: \<level\>**

这个指令指定日志语句和操作统计信息应该发送到syslog的级别。您必须配置OpenLDAP——enable-debug(默认值)才能使其工作，除了始终启用的两个统计级别

**olcReferral \<URI\>**

这个指令指定了当slapd找不到本地数据库来处理请求时要返回的引用。

```
Example:
        olcReferral: ldap://root.openldap.org
```

**例子：**
```
dn: cn=config
objectClass: olcGlobal
cn: config
olcIdleTimeout: 30
olcLogLevel: Stats
olcReferral: ldap://root.openldap.org
```

##### cn=module

如果在配置slapd时启用了对动态加载模块的支持，则cn=module条目可以用来指定要加载的模块集。模块条目必须有objectClass: olcModuleList。

**olcModuleLoad: \<filename\>**

指定要加载的动态可加载模块的名称。文件名可以是绝对路径名，也可以是简单文件名。非绝对名称会在olcModulePath指令指定的目录中搜索。

**olcModulePath: \<pathspec\>**

指定要搜索可加载模块的目录列表。通常路径以冒号分隔，但这取决于操作系统。

**样例：**
```
dn: cn=module{0},cn=config
objectClass: olcModuleList
cn: module{0}
olcModuleLoad: /usr/local/lib/smbk5pwd.la

dn: cn=module{1},cn=config
objectClass: olcModuleList
cn: module{1}
olcModulePath: /usr/local/lib:/usr/local/lib/slapd
olcModuleLoad: accesslog.la
olcModuleLoad: pcache.la
```

##### cn=schema

cn=schema条目包含在slapd中硬编码的所有模式定义。因此，这个条目中的值是由slapd生成的，因此不需要在配置文件中提供模式值。但是，仍然必须定义条目，以作为下面添加用户定义模式的基础。模式条目必须有objectClass: olcSchemaConfig.

**olcAttributeTypes: \<RFC4512 Attribute Type Description\>**

这个指令定义了一个属性类型。

**olcObjectClasses: \<RFC4512 Object Class Description\>**

这个指令定义了一个对象类。

**样例：**

```
dn: cn=schema,cn=config
objectClass: olcSchemaConfig
cn: schema

dn: cn=test,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: test
olcAttributeTypes: ( 1.1.1
  NAME 'testAttr'
  EQUALITY integerMatch
  SYNTAX 1.3.6.1.4.1.1466.115.121.1.27 )
olcAttributeTypes: ( 1.1.2 NAME 'testTwo' EQUALITY caseIgnoreMatch
  SUBSTR caseIgnoreSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.44 )
olcObjectClasses: ( 1.1.3 NAME 'testObject'
  MAY ( testAttr $ testTwo ) AUXILIARY )
```

#### Openldap和phpldapadmin镜像运行

先查找镜像，然后拉取镜像
```
$ docker search openldap
$ docker pull openldap

$ docker search phpldapadmin
$ docker pull phpldapadmin
```
镜像就都到了我们本地，然后运行镜像

```
$ docker run --name ldap-service -p 389:389 -p 636:636 --env PHPLDAPADMIN_HTTPS=false --env PHPLDAPADMIN_LDAP_HOSTS=127.0.0.1 --hostname ldap-service --detach osixia/openldap

$ docker run --name phpldapadmin-service -p 8080:80 -p 6443:443 --hostname phpldapadmin-service --link ldap-service:ldap-host --env PHPLDAPADMIN_LDAP_HOSTS=ldap-host --detach osixia/phpldapadmin:0.9.0
```

然后访问https://127.0.0.1:6443/，进入登录页面，用户名密码分别是：
```
"Login DN: cn=admin,dc=example,dc=org"
"Password: admin"
```

![upload successful](/images/pasted-109.png)

#### phpldapadmin配置与使用

配置文件在：
```
$ /container/environment/XX-somedir
```
（XX < 99，因此它们将在默认环境文件之前处理）而不是直接链接到

假如你的配置文件在/data/environment/my-env.yaml中，挂载目录到容器里/container/environment/01-custom/env.yaml：
```
$ docker run --volume /data/environment/my-env.yaml:/container/environment/01-custom/env.yaml \
--detach osixia/phpldapadmin:0.9.0
```

进入容器可以用命令：
```
$ docker exec -it phpldapadmin-service bash
```













#### 附

[参考安装网站](https://www.openldap.org/doc/admin26/quickstart.html)  
[配置参考网站](https://www.openldap.org/doc/admin26/slapdconf2.html)
[phpLDAPadmin github地址](https://github.com/osixia/docker-phpLDAPadmin)
[openldap github地址](https://github.com/osixia/docker-openldap)
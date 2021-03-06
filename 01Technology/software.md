
[TOC]

## Project
### Design
#### API
1. lower <, floor <=, ceiling >=, higher >
2. 返回list时带上下一条,然后请求数据时带上第一条记录 查询用它做条件减少翻页

### log config

`-Dlog4j.debug=true` print debug log for log4j on startup

### Tomcat build
#### Code
Entry point is Bootstrap
set java_home=C:\ProgramFiles\Java\jdk1.6.0_45
set java_home=C:\ProgramFiles\Java\jdk1.7.0_21
1. read BUILDING.txt (it needs Patch for version tomcat 8.0)
	1.1 (2.1) Checkout or obtain the source code for Tomcat 6.0
	1.2 set base.path=C:/ProgramFiles/Apache/usr/share/java in file build.properties.default
	1.3 (2.2) Building: ant download then ant
2. read RUNNING.txt
	2.1 (3) Start Up Tomcat
	2.2 change to folder output\build\bin, comment line
		"if not "%CATALINA_HOME%" == "" goto gotHome" in files startup.bat and catalina.bat
	2.3 (3.1)run bat startup.bat
3. log configuration
	3.1 apache-tomcat-6.0.36\conf\logging.properties
	```
	############################################################
	# Added by Shang Pu
	# refer to http://www.student.lu.se/docs/logging.html
	############################################################
	#The default logging.properties specifies a ConsoleHandler for routing logging to stdout and also a FileHandler. A handler's log level threshold can be set using SEVERE, WARNING, INFO, CONFIG, FINE, FINER, FINEST or ALL. The logging.properties shipped with JDK is set to INFO. You can also target specific packages to collect logging from and specify a level. Here is how you would set debugging from Tomcat. You would need to ensure the ConsoleHandler's level is also set to collect this threshold, so FINEST or ALL should be set. Please refer to Sun's java.util.logging documentation for the complete details.
	#org.apache.catalina.level=FINE
	```

#### Configuration
URL with Chinese URIEncoding="UTF-8"
```
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" URIEncoding="UTF-8" />
```
for tomcat 7 set Xmx
echo 'export JAVA_OPTS="-Xms1024M -Xmx2048M"' > $CATALINA_HOME/bin/setenv.sh

### Bug report for Tomcat
```
+---------------------------------------------------------------------------+
| Bugzilla Bug ID                                                           |
|     +---------------------------------------------------------------------+
|     | Status: UNC=Unconfirmed NEW=New         ASS=Assigned                |
|     |         OPN=Reopened    VER=Verified    (Skipped Closed/Resolved)   |
|     |   +-----------------------------------------------------------------+
|     |   | Severity: BLK=Blocker CRI=Critical  REG=Regression  MAJ=Major   |
|     |   |           MIN=Minor   NOR=Normal    ENH=Enhancement TRV=Trivial |
|     |   |   +-------------------------------------------------------------+
|     |   |   | Date Posted                                                 |
|     |   |   |          +--------------------------------------------------+
|     |   |   |          | Description                                      |
|     |   |   |          |                                                  |
|45392|New|Nor|2008-07-14|No OCSP support for client SSL verification       |
|46179|Opn|Maj|2008-11-10|apr ssl client authentication                     |
|48655|Inf|Nor|2010-02-02|Active multipart downloads prevent tomcat shutdown|
```

### lucene
run test
set java VM parameter "-ea"	to enable

### Spring
https://src.springframework.org/svn/spring-samples/
consider running with the -a flag to avoid evaluating other subprojects you depend on.
For example, if you're iterating on changes in spring-webmvc,
cd spring-webmvc
run ../gradlew -a build to tell gradle to evaluate and build only that subproject.

#### Spring artifact versioning
```
3.1.0.RELEASE
| | | | - version type
| | | --- maintenance version
| | ----- major version
| ------- project generation
```
3.1.0.BUILD-SNAPSHOT - nightly snapshot of 3.1.0 development
3.1.0.M1             - first milestone release toward 3.1.0 GA
3.1.0.M2             - second milestone release toward 3.1.0 GA
3.1.0.RC1            - first release candidate toward 3.1.0 GA
3.1.0.RC2            - second release candidate toward 3.1.0 GA
3.1.0.RELEASE        - final GA (generally available) release of 3.1.0
3.1.1.BUILD-SNAPSHOT - nightly snapshot of the 3.1.1 maintenance release
3.1.1.RELEASE        - final GA release of 3.1.1

#### Gradle command:
https://github.com/SpringSource/spring-framework/wiki/Gradle-build-and-release-FAQ
set DEFAULT_JVM_OPTS=-Xms768m -Xmx1024m
set GRADLE_OPTS= %GRADLE_OPTS%
gradlew eclipse
gradlew build

## Operating System
### Windows
#### Misc.
replace one line in notepad++ which contain REPLEACH_THIS_LINE	`^.*REPLEACH_THIS_LINE.*$\r\n`
F4 重复上一步操作，比如，插入行、设置格式等等频繁的操作, 公式里面切换绝对引用，直接点选目标，按F4轮流切换
Alt+Shift+D    - Current date;
Alt+Shift+T    - Current time.
start cmd /k
type: Windows Displays the contents of a text file or files.
nslookup http://wsyc.lqwang.com/
tracert http://haijia.bjxueche.net/

## Network tools
### putty显示中文
在window-〉Appearance-〉Translation中，Received data assumed to be in which character set 中,把Use font encoding改为UTF-8.
如果经常使用,把这些设置保存在session里面.

### Proxy configuration
file://path
	On Windows, you have to use that syntax: file://C:/proxy.pac
	On Unix, you have to use that syntax: file:///path/to/proxy.pac

https://pac.itzmx.com/abc.pac

## Editor

## Data Security
### Encrypt
#### veracrypt
https://veracrypt.codeplex.com

## Classify by Project Development Phase
### 01 Project Management
develop, test, staging, production environment

### 02 Requirements
### 03 Design
#### swagger
http://swagger.io
A Powerful Interface to your API
https://github.com/swagger-api/swagger-ui

### 04 Build
#### Java run
java $JAVA_OPTS -Xms1024m -Xmx1024m -XX:+UseParallelOldGC -XX:MaxPermSize=256m -verbose:gc
-Xms2m -Xmx8m -Djava.rmi.server.hostname=AMRGROLL3DK364 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=5000 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false

#### RabbitMQ

##### [Troubleshooting Guidance](https://www.rabbitmq.com/troubleshooting.html)

crash file /usr/local/rabbitmq/var/lib/rabbitmq/erl_crash.dump

##### Basic command

``` bash
service rabbitmq-server statrt
service rabbitmq-server stop
service rabbitmq-server status
rabbitmqctl status
rabbitmqctl environment # 环境信息

rabbitmqctl list_vhosts
rabbitmqctl list_queues -p <vhost>
rabbitmqctl list_consumers -p <vhost>
rabbitmqctl list_exchanges -p push-center
rabbitmqctl list_user_permissions username
rabbitmqctl purge_queue queue.name.development -p <vhost>
curl -X DELETE -i -u guest:guest "http://localhost:15672/api/queues/local/gongzhonghao.refreshAccessToken.queue.name.development"
```

查看用户 `rabbitmqctl list_users`
By default, RabbitMQ have a user named guest with password guest. We will create own administrator account on RabbitMQ server, change password :

``` bash
rabbitmqctl add_user admin password
rabbitmqctl set_user_tags admin administrator
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"

rabbitmqctl add_vhost vhost_name
rabbitmqctl set_permissions -p vhost_name admin ".*" ".*" ".*"
```

RabbitMQ has a web management console. To enable web management console run : `rabbitmq-plugins enable rabbitmq_management`
`http://localhost:15672/   guest / guest`

`curl -i -u guest:guest "http://localhost:15672/api/overview"`  
`curl -i -u guest:guest "http://localhost:15672/api/queues/push-center/push.schedule.push.queue.name.publish"`  

The OS limits are controlled via a configuration file at `/etc/systemd/system/rabbitmq-server.service.d/limits.conf`, for example:

##### Installation

install erlang  

``` bash
wget https://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb
dpkg -i erlang-solutions_1.0_all.deb
apt update
apt install erlang erlang-nox
```

install rabbitmq `apt-get install rabbitmq-server`  

#### memcached

##### options

-p <num> 监听的TCP端口 (缺省: 11211)

-d 以守护进程方式运行Memcached

-u <username> 运行Memcached的账户，非root用户

-m <num> 最大的内存使用, 单位是MB，缺省是 64 MB

-c <num> 软连接数量, 缺省是 1024

-v 输出警告和错误信息

-vv 打印客户端的请求和返回信息

-h 打印帮助信息

-i 打印memcached和libevent的版权信息

运行 Memcached 目标：使用11211端口、最大占用512M内存、1024个软连接, 不监听UDP `-U 0`，输出客户端请求，以守护进程方式运行

`/usr/local/bin/memcached -p 11211 -d -u root -m 512 -c 1024 -U 0 -vvv`

### 05 Test

UAT: User Acceptance Test
### 06 Deploy
#### Data security 数据安全工具DRBD

#### 高性能集群软件 Keepalived

#### 高并发负载均衡软件 HAProxy

#### 构建高性能的 MySQL 集群系统
##### 通过KeepAlived搭建 MySQL双主模式的高可用集群系统
##### 通过MMM构建MySQL高可用集群系统
#####  MySQL读写分离解决方案
通过amoeba 实现MySQL读写分离
通过keepalived构建高可用的amoeba服务

#### Distributed Configuration Management Platform(分布式配置管理平台)
https://github.com/knightliao/disconf

#### Commons Configuration
http://commons.apache.org/proper/commons-configuration/
配置管理和部署使用chef/puppet/ansible，密码放在加密的data bag里（如chef）
密码管理通用服务vault和keywhiz, vault是用golang写的，github上的like比java写的keywhiz多些，同时它又是consul的东家hashicorp做

#### Web Server
##### Node
npm install
npm start
nmp stop
cp local-sample.js local.js
npm install pm2 -g
pm2 start bin/www
pm2 stop bin/www
pm2 list
pm2 delete id

### 07 Support
Django的出现让运维非常快速方便地开发部署自动化工具

#### Monitor
##### Zabbix
Zabbix is an open source monitoring software

##### Ganglia

##### 基于 nagios 的分布式监控平台 centreon

##### Java HeartBeat
https://www.oschina.net/news/62034/java-heartbeat-0-4
[git.oschina](http://git.oschina.net/mkk/HeartBeat)
[下载链接](http://git.oschina.net/mkk/HeartBeat/raw/V-0.4/dist/HeartBeat-0.4.zip)
[在线测试](http://andaily.com/hb)

##### JavaMelody
是一款用来监控Java应用或服务器的监控统计工具，以图表形式展示监控数据
[Home](https://github.com/javamelody/javamelody/wiki)

##### clamAV 杀毒软件
[ClamAV from Ubuntu](https://help.ubuntu.com/community/ClamAV)
##### usage
`freshclam` 更新病毒库
`clamdscan /path/to/file` 扫描病毒
`clamdscan--remove /path/to/file` 删掉病毒文件

##### installation and startup
1. vi /etc/yum.repos.d/dag.repo
```

	#Dag RPM Repository Start
	[dag]
	name=Dag RPM Repository for RHEL4
	baseurl=http://ftp.riken.jp/Linux/dag/redhat/el4/en/$basearch/dag/
	enabled=1
	gpgcheck=1
	#Dag RPM Repository End
```
2. `yum -y install clamd`
3. `service clamd start`
4. `service clamd status`


#### 运维工具组合的进化
##### 命令执行与配置管理
Ansible, SaltStack, Puppet  

##### 持续交付与代码
Jenkins, 国内Coding.net, GitCafe，Git@OSC的兴起, GitLab的进步与稳定
##### ELK生态的成熟
提供日志收集，分析，和实时搜索，与可视化监控
##### 应用监控 APM

##### 国内开源 open-falcon

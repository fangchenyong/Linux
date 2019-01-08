# Centos下安装JDK三种方法

### 方法一：手动下载JDK压缩包或者本地上传解压，然后设置环境变量

#### 1. 在/usr/目录下创建java目录

```shell
[root@localhost ~]# mkdir/usr/java
[root@localhost ~]# cd /usr/java
```

#### 2. 下载jdk,然后解压

```shell
[root@localhost java]# curl -O （jdk下载链接）
[root@localhost java]# tar -zxvf jdk-8u192-linux-x64.tar.gz
```

#### 3. 设置环境变量

```shell
[root@localhost java]# vim /etc/profile
```

> 在profile中最后添加如下内容:

```properties
JAVA_HOME=/usr/java/jdk1.8.0_192
JRE_HOME=/usr/java/jdk1.8.0_192/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
```

> 让修改生效:

```shell
[root@localhost java]# source /etc/profile
```

>  若因修改环境变量导致系统出现command not found错误,可尝试以下方法

```shell
#编辑环境变量文件
[root@localhost java]# /usr/bin/vim /etc/profile
#修改或删除原先配置文件 :wq 保存退出 执行export
[root@localhost java]# export PATH=/usr/bin:/usr/sbin:/bin:/sbin
#重新执行使更改立即生效
[root@localhost java]# source /etc/profile
```

#### 4. 验证JDK有效性

```shell
[root@VM_0_5_centos jdk1.8.0_192]# java -version
java version "1.8.0_192"
Java(TM) SE Runtime Environment (build 1.8.0_192-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.192-b12, mixed mode)
```

### 方法二：使用yum命令在线安装JDK

#### 1. 查看yum库中都有哪些jdk版本，全是openjdk

```shell
[root@VM_0_5_centos jdk1.8.0_192]# yum search java|grep jdk
ldapjdk-javadoc.noarch : Javadoc for ldapjdk
java-1.6.0-openjdk.x86_64 : OpenJDK Runtime Environment
...
java-1.7.0-openjdk.x86_64 : OpenJDK Runtime Environment
...
java-1.8.0-openjdk.i686 : OpenJDK Runtime Environment
java-1.8.0-openjdk.x86_64 : OpenJDK Runtime Environment
java-1.8.0-openjdk-accessibility.i686 : OpenJDK accessibility connector
java-1.8.0-openjdk-accessibility.x86_64 : OpenJDK accessibility connector
java-1.8.0-openjdk-accessibility-debug.i686 : OpenJDK accessibility connector
java-1.8.0-openjdk-accessibility-debug.x86_64 : OpenJDK accessibility connector
java-1.8.0-openjdk-debug.i686 : OpenJDK Runtime Environment with full debug on
java-1.8.0-openjdk-debug.x86_64 : OpenJDK Runtime Environment with full debug on
java-1.8.0-openjdk-demo.i686 : OpenJDK Demos
java-1.8.0-openjdk-demo.x86_64 : OpenJDK Demos
java-1.8.0-openjdk-demo-debug.i686 : OpenJDK Demos with full debug on
java-1.8.0-openjdk-demo-debug.x86_64 : OpenJDK Demos with full debug on
java-1.8.0-openjdk-devel.i686 : OpenJDK Development Environment
java-1.8.0-openjdk-devel.x86_64 : OpenJDK Development Environment
java-1.8.0-openjdk-devel-debug.i686 : OpenJDK Development Environment with full
java-1.8.0-openjdk-devel-debug.x86_64 : OpenJDK Development Environment with
java-1.8.0-openjdk-headless.i686 : OpenJDK Runtime Environment
java-1.8.0-openjdk-headless.x86_64 : OpenJDK Runtime Environment
java-1.8.0-openjdk-headless-debug.i686 : OpenJDK Runtime Environment with full
java-1.8.0-openjdk-headless-debug.x86_64 : OpenJDK Runtime Environment with full
java-1.8.0-openjdk-javadoc.noarch : OpenJDK API Documentation
java-1.8.0-openjdk-javadoc-debug.noarch : OpenJDK API Documentation for packages
java-1.8.0-openjdk-javadoc-zip.noarch : OpenJDK API Documentation compressed in
java-1.8.0-openjdk-javadoc-zip-debug.noarch : OpenJDK API Documentation
java-1.8.0-openjdk-src.i686 : OpenJDK Source Bundle
java-1.8.0-openjdk-src.x86_64 : OpenJDK Source Bundle
java-1.8.0-openjdk-src-debug.i686 : OpenJDK Source Bundle for packages with
java-1.8.0-openjdk-src-debug.x86_64 : OpenJDK Source Bundle for packages with
java-11-openjdk.i686 : OpenJDK Runtime Environment 11
...
```

#### 2. 选择版本进行安装

 ```shell
#选择1.8版本进行安装,安装完之后，默认的安装目录是在: /usr/lib/jvm/
[root@localhost ~]# yum install java-1.8.0-openjdk
 ```

#### 3. 设置环境变量 ，使修改立即生效，查看是否安装成功参考方法一

### 方法三：使用rpm安装

#### 1. 其他步骤具体参考上述方法

```shell
[root@localhost  ~]# rpm -ivh jdk-7u79-linux-x64.rpm
```




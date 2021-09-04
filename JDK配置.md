# JDK配置

```bash
tar xf jdk-8u301-linux-x64.tar.gz  -C /usr/local/
cd /usr/local/jdk1.8.0_301/
cp /etc/profile{,.bak} && ll /etc/profile*
echo 'export JAVA_HOME=/usr/local/jdk1.8.0_301' >> /etc/profile
echo 'export JRE_HOME=${JAVA_HOME}/jre' >> /etc/profile
echo 'export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib' >> /etc/profile
echo 'export PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin:$PATH' >> /etc/profile
source /etc/profile
[root@node1 ~]# java -version
java version "1.8.0_301"
Java(TM) SE Runtime Environment (build 1.8.0_301-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.301-b09, mixed mode)
[root@node1 ~]#
```


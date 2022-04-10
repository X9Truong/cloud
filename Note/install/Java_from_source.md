### Install java from source

```
cd /opt/
wget tar -zxvf jdk-8u151-linux-x64.tar.gz
mv jdk-8u151-linux-x64 jdk


vi /etc/profile
JAVA_HOME=/opt/jdk
export JAVA_HOME
CLASSPATH=.:$JAVA_HOME/lirootb:$JAVA_HOME/jre/lib
export CLASSPATH
JRE=$JAVA_HOME/jre
export JRE
PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH


source /etc/profile
java -version

ln -s /opt/zulu-11-azure-jdk_11.41.23-11.0.8-linux_x64/bin/java /usr/bin/java
```

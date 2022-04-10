### Install jenkins centos 7

- Config OS https://github.com/anhbka/bk/tree/master/Note/Turning.md

* Install java

`yum -y install java`

Check version: `java -version`

Expect: 
```
openjdk version "1.8.0_161"
OpenJDK Runtime Environment (build 1.8.0_161-b14)
OpenJDK 64-Bit Server VM (build 25.161-b14, mixed mode)
```

* Install jenkins

```
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum -y install jenkins
```

* Start jenkins

```
systemctl start jenkins
systemctl enable jenkins
systemctl status jenkins
```

Test port jenkins running: `netstat -atnp | grep 8080`

Expect: 
```
[root@vm2 ~]# netstat -atnp | grep 8080
tcp6       0      0 :::8080                 :::*                    LISTEN      15386/java
```

* Setting jenkins

- Access website with password admin `cat /var/lib/jenkins/secrets/initialAdminPassword`:

`http://192.168.30.130:8080/`
### ETCD

Etcd là một CSDL dạng key-value được dùng để lưu dữ liệu về hệ thống, tài nguyên và cấu hình của kubenetes cluster. Khi bạn tạo mới các ứng dụng bao gồm những deployment, pod, service.. thì những thông tin định nghĩa này sẽ được lưu vào etcd.

Etcd được chạy dưới dạng cluster và số lượng member trong cluster là lẻ. Theo tài liệu của etcd thì recommand cho production là nên cài 5 node etcd để đảm bảo tính sẵn sàng (availability). 

Để sử dụng thì ta cần cài đặt và cấu hình certificate cho nó có thể kết nối được tới etcd cluster. Cú pháp sử dụng etcdctl như sau:

```
ETCDCTL_API=3 etcdctl <etcd-command> \
--endpoints=https://127.0.0.1:2379 \
--cacert=<trusted-ca-file> \
--cert=<cert-file> \
--key=<key-file>
```

### Cài đặt etcdctl

```
curl -L https://github.com/etcd-io/etcd/releases/download/v3.5.5/etcd-v3.5.5-linux-amd64.tar.gz -o etcd-v3.5.5-linux-amd64.tar.gz
tar xzvf etcd-v3.5.5-linux-amd64.tar.gz
cd etcd-v3.5.5-linux-amd64
cp -rp etcd* /usr/bin/
chown -R root:root /usr/bin/etcdctl
etcdctl version
```

* Bash shell

https://github.com/etcd-io/etcd/releases/

```
ETCD_VER=v3.5.5

# choose either URL
GOOGLE_URL=https://storage.googleapis.com/etcd
GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
DOWNLOAD_URL=${GOOGLE_URL}

rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
rm -rf /tmp/etcd-download-test && mkdir -p /tmp/etcd-download-test

curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd-download-test --strip-components=1
rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz

/tmp/etcd-download-test/etcd --version
/tmp/etcd-download-test/etcdctl version
```

```
yum install epel-release -y
yum install jq -y

kubectl get pod -n kube-system
NAME                             READY   STATUS    RESTARTS        AGE
coredns-565d847f94-g8f2h         1/1     Running   11 (134m ago)   35d
coredns-565d847f94-p2h92         1/1     Running   11 (134m ago)   35d
etcd-master                      1/1     Running   12 (134m ago)   35d
kube-apiserver-master            1/1     Running   12 (134m ago)   35d
kube-controller-manager-master   1/1     Running   12 (134m ago)   35d
kube-proxy-5cs74                 1/1     Running   11 (134m ago)   35d
kube-proxy-9vd7t                 1/1     Running   11 (134m ago)   35d
kube-proxy-xv97d                 1/1     Running   11 (134m ago)   35d
kube-scheduler-master            1/1     Running   12 (134m ago)   35d
weave-net-8cwpm                  2/2     Running   20 (134m ago)   34d
weave-net-dlp77                  2/2     Running   21 (133m ago)   34d
weave-net-lslzl                  2/2     Running   20 (134m ago)   34d

kubectl -n kube-system get pod kube-apiserver-master -o=jsonpath='{.spec.containers[0].command}' |jq |grep etcd
  "--etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt",
  "--etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt",
  "--etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key",
  "--etcd-servers=https://127.0.0.1:2379",
```

Như vậy ta đã có thông tin các etcd endpoints và đường dẫn tới các file cert trên node master1. Giờ trên node cicd này, ở thư mục chúng ta từ tạo (mkdir -p /opt/etcd_backup) ta lần lượt tạo ra 3 file:
```
ca.crt
apiserver-etcd-client.crt
apiserver-etcd-client.key
```

* Backup etcd


```
export ETCDCTL_CACERT=/opt/etcd_backup/etcd_backup/ca.pem
export ETCDCTL_CERT=/opt/etcd_backup/etcd_backup/cert.pem
export ETCDCTL_KEY=/opt/etcd_backup/etcd_backup/key.pem
export ETCDCTL_API=3

etcdctl snapshot save /opt/backup/etcd.backup
etcdctl snapshot status /opt/backup/etcd.backup --wirte-out=table

+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 9dbfc909 |   236505 |        896 |     5.6 MB |
+----------+----------+------------+------------+

```

* Restore etcd 

- Run all node master

```
mkdir -p /opt/backup/yaml
mv /etc/kubernetes/manifests/*.yaml /opt/backup/yaml
cat /etc/kubernetes/manifests/etcd.yaml | grep hostPath -C2
```


Lưu ý: Sau khi thực hiện bước này thì lệnh kubectl cũng sẽ không hoạt động nữa do ta đã stop hết kube-api-server rồi.

Các bạn có thể verify lại bằng cách kiểm tra các container của api, controller và scheduler đều đã không còn:

`docker ps -a |egrep "api|schedule|control"`

```
mv /var/lib/etcd/member/ /var/lib/etcd/member.bak
cd /opt/backup/
etcdctl snapshot restore etcd.backup
cd default.etcd/ # copy folder member to all node master or run command bellow
etcdctl snapshot restore etcd.backup --data-dir=/var/lib/etcd

```

* Start các control plane instance

Run all node master:

```
mv /opt/backup/yaml/*.yaml /etc/kubernetes/manifests/
systemctl restart docker
docker ps |egrep "api|schedule|control"
kubectl get nodes
```
















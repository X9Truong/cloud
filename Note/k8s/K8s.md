### How to Install a Kubernetes Cluster on CentOS 7

```
Multiple servers running Centos 7 (1 Master Node, 2 Worker Nodes). It is recommended that your Master Node have at least 2 CPUs, though this is not a strict requirement.
Internet connectivity on all your nodes. We will be fetching Kubernetes and docker packages from the repository. Equally, you will need to make sure that the yum package manager is installed by default and can fetch packages remotely.
You will also need access to an account with sudo or root privileges. In this tutorial, I will be using my root account.
```

* Installation of Kubernetes Cluster on Master-Node

- Step 1: Prepare Hostname, Firewall and SELinux

```
hostnamectl set-hostname master-node
cat <<EOF>> /etc/hosts
10.128.0.27 master-node
10.128.0.29 node-1 worker-node-1
10.128.0.30 node-2 worker-node-2
EOF

setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
reboot

modprobe br_netfilter

echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

```

* Install Kubeadm and Docker

```
yum install kubeadm docker -y 

systemctl enable kubelet
systemctl start kubelet
systemctl enable docker
systemctl start docker
swapoff -a
```

*  Update Iptables Settings
```
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

* Initializing Kubernetes master is a fully automated process that is managed by the “kubeadm init“ command which you will run.

```
kubeadm init 
kubeadm init --apiserver-advertise-address=192.168.33.101
sudo kubeadm init --apiserver-advertise-address=192.168.30.128 --pod-network-cidr=10.244.0.0/16

kubeadm join 192.168.30.128:6443 --token gyjp2p.gxq4gj9neveitl4y --discovery-token-ca-cert-hash sha256:a7acb1f8e6769d1adaf5d09041d67871b753536135350ed325a8cccbc926eba6
```
```
vim create_pod.yml

apiVersion: v1
kind: Pod
metadata: 
	name: hello-pod
spec: 
		containers:
		- name: hell-nginx
		  image: nginx
		  
```

`kubectl apply -f create_pod.yml`

`kubectl describe pod`

`kubectl get pod`

* Deployment controller -> Replicaset controller -> pod -> nginx

`kubectl run hello-nginx --image nginx`

`kubectl get all`

`kubectl scale deployment hell-nginx --replicas 5`

### Creating and exploring an nginx deployment 

```
vim apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

* Create a Deployment based on the YAML file:

` kubectl apply -f https://k8s.io/examples/application/deployment.yaml`

* Display information about the Deployment:

` kubectl describe deployment nginx-deployment`

* List the Pods created by the deployment:

`kubectl get pods -l app=nginx`

* Display information about a Pod:

`kubectl describe pod <pod-name>`

### Updating the deployment

```
vim apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.16.1 # Update the version of nginx from 1.14.2 to 1.16.1
        ports:
        - containerPort: 80
```
* Apply the new YAML file:

`kubectl apply -f https://k8s.io/examples/application/deployment-update.yaml`

* Watch the deployment create pods with new names and delete the old pods:

`kubectl get pods -l app=nginx`

### Scaling the application by increasing the replica count

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 4 # Update the replicas from 2 to 4
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
		
`kubectl apply -f https://k8s.io/examples/application/deployment-scale.yaml`

* Deleting a deployment

`kubectl delete deployment nginx-deployment`


```
docker login
anhbka
docker build -t anhbka .
docker tag anhbka:latest anhbka/hello-kube
docker push  anhbka/hello-kube:latest
docker pull anhbka/hello-kube
```

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-kube-testing
  labels:
    enviroment: testing # label with key is enviroment and value is testing
    project: kubernetes-series
spec:
  containers:
    - image: anhbka/hello-kube
      name: hello-kube
      ports:
        - containerPort: 3000
          protocol: TCP

---
apiVersion: v1
kind: Pod
metadata:
  name: hello-kube-staging
  labels:
    enviroment: staging # label with key is enviroment and value is staging
    project: kubernetes-series
spec:
  containers:
    - image: anhbka/hello-kube
      name: hello-kube
      ports:
        - containerPort: 3000
          protocol: TCP

---
apiVersion: v1
kind: Pod
metadata:
  name: hello-kube-production
  labels:
    enviroment: production # label with key is enviroment and value is production
    project: kubernetes-series
spec:
  containers:
    - image: anhbka/hello-kube
      name: hello-kube
      ports:
        - containerPort: 3000
          protocol: TCP

```		  


```
kubectl apply -f hello-kube.yaml

kubectl get pod --show-labels

kubectl get pod -L enviroment

kubectl get pod -l enviroment=production

kubectl delete -f hello-kube.yaml

```


```
kubectl get pod --namespace kube-system
kubectl get ns
kubectl get pod -n testing
kubectl delete pod hello-kube-testing -n testing
kubectl delete ns testing
```

### ReplicationControllers

ReplicationControllers là một resource mà sẽ tạo và quản lý pod, và chắc chắn là số lượng pod nó quản lý không thay đổi và keep running. ReplicationControllers sẽ tạo số lượng pod bằng với số ta chỉ định ở thuộc tính replicas và quản lý pod thông qua labels của pod

Tại sao ta nên dùng ReplicationControllers để chạy pod?
Chúng ta đã biết pod nó sẽ giám sát container và tự động restart lại container khi nó failed

```
vim replicationcontroller.yml

apiVersion: v1 # Descriptor conforms to version v1 of Kubernetes API
kind: Pod # Select Pod resource
metadata:
  name: hello-kube # The name of the pod
spec:
  containers:
    - image: anhbka/hello-kube # Image to create the container
      name: hello-kube # The name of the container
      ports:
        - containerPort: 3000 # The port the app is listening on 
          protocol: TCP
```
 
```
kubectl apply -f replicationcontroller.yml
replicationcontroller/hello-rc created
root@k8s1:/opt/javascript# kubectl get rc
NAME       DESIRED   CURRENT   READY   AGE
hello-rc   2         2         2       10s
root@k8s1:/opt/javascript# kubectl get rc
NAME       DESIRED   CURRENT   READY   AGE
hello-rc   2         2         2       23s
root@k8s1:/opt/javascript# kubectl get pod
NAME                        READY   STATUS    RESTARTS   AGE
hello-rc-qnhmx              1/1     Running   0          33s
hello-rc-x6d6w              1/1     Running   0          33s
my-hello-5698bd5fbb-8rn5c   1/1     Running   0          33m
root@k8s1:/opt/javascript# kubectl delete pod hello-rc-qnhmx
pod "hello-rc-qnhmx" deleted
```

### Sử dụng ReplicaSets thay thế RC

Đây là một resource tương tự như RC, nhưng nó là một phiên bản mới hơn của RC và sẽ được sử dụng để thay thế RC.

Sử dụng DaemonSets để chạy chính xác một pod trên một worker node

Ứng dụng của thằng DaemonSets này sẽ được dùng trong việc logging và monitoring. Lúc này thì chúng ta sẽ chỉ muốn có một pod monitoring ở mỗi node. Và ta cũng có thể đánh label vào trong một thằng woker node bằng cách sử dụng câu lệnh

`kubectl label nodes <your-node-name> disk=ssd`

Sau đó ta có thể chỉ định thêm vào config của DaemonSets ở cột nodeSelector với disk=ssd. Chỉ deploy thằng pod trên node có ổ đĩa ssd. Đây là config ví dụ

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ssd-monitor
spec:
  selector:
    matchLabels:
      app: ssd-monitor
  template:
    metadata:
      labels:
        app: ssd-monitor
    spec:
      nodeSelector:
        disk: ssd
      containers:
        - name: main
          image: luksa/ssd-monitor
```

### Services: expose traffic cho pod

<img src="/Note/img/sv.png">	

ClusterIP:

Đây là loại service sẽ tạo một IP và local DNS mà sẽ có thể truy cập ở bên trong cluster, không thể truy cập từ bên ngoài, được dùng chủ yếu cho các Pod ở bên trong cluster dễ dàng giao tiếp với nhau.

Ví dụ như sau, ta có một application cần xài redis, ta sẽ deploy một Pod redis và application của ta cần kết nối với Pod redis đó. Với redis thì connection string của nó sẽ có dạng như sau `redis://<host-name>:<port>`, vậy ứng dụng ta sẽ kết nối với redis thế nào? Sau đây ta sẽ làm ví dụ tạo Pod redis và Service cho nó.

Tạo một file tên là hello-service.yaml
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - image: redis # redis image
        name: redis
        ports:
          - containerPort: 6379

---
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  selector:
    app: redis # label selectors Pod redis
  ports:
    - port: 6379 # port of the serivce
      targetPort: 6379 # port of the container that service will forward to 

```
`kubectl apply -f hello-service.yaml`

<img src="/Note/img/IPcluster.png">	  

Bây redis của chúng ta sẽ có địa chỉ truy cập như sau redis://10.100.255.140:6379. Ta có thể thấy hostname của service của chúng ta ở đây là 10.100.255.140, ta có thể dùng địa chỉ này để các ứng ta deploy về sau truy cập được tới redis host.

Vậy nếu ta có các ứng dụng đã chạy sẵn, thì làm sao ta kết nối tới host này được, không lẻ chúng ta phải update lại code? Thì để giải quyết vấn đề này, thay vì truy cập redis host thằng IP, ta có thể truy cập thông qua DNS, kubernes có cung cấp cho chúng ta một local DNS bên trong cluster, giúp chúng ta có thể connect được tới host ta muốn thông qua DNS. Ta có thể connect redis với địa chỉ sau `redis://redis:6379`, với host name là tên của Service chúng ta đặt trong trường metadata.

<img src="/Note/img/IPcluster2.png">	 
<img src="/Note/img/IPcluster3.png">	 
Test thử service. Các bạn có thể tạo Pod để test bằng file config

```
apiVersion: v1
kind: Pod
metadata:
  name: application
spec:
  containers:
    - image: 080196/hello-redis
      name: application
```
Hoặc tạo bằng cli cho nhanh: `kubectl run hello-redis --image=080196/hello-redis`


Đây là code của application trong 080196/hello-redis image	  

```
const redis = require("redis");

const client = redis.createClient("redis://redis:6379")

client.on("connect", () => {
  console.log("Connect redis success");
})

```
Kiểm tra log của pod, nếu in ra Connect redis success thì chúng ta đã kết nối được với redis host bằng dns
kubectl logs hello-redis

`kubectl logs hello-redis`

<img src="/Note/img/IPcluster4.png">


NodePort:


Đây là cách đầu tiên để expose Pod cho client bên ngoài có thể truy cập vào được Pod bên trong cluster. Giống như ClusterIP, NodePort cũng sẽ tạo endpoint có thể truy cập được bên trong cluster bằng IP và DNS, đồng thời nó sẽ sử dụng một port của toàn bộ worker node để client bên ngoài có thể giao tiếp được với Pod của chúng ta thông qua port đó. NodePort sẽ có range mặc định từ 30000 - 32767. Ví dụ:
```
apiVersion: v1
kind: Service
metadata:
  name: hello
spec:
  selector:
    app: hello-kube
  type: NodePort # type NodePort
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30123 # port of the worker node
```

<img src="/Note/img/nodeport.png">

Client có thể gửi request tới Pod bằng địa chỉ 130.211.97.55:30123 hoặc 130.211.99.206:30123. Và application bên trong cluster có thể gửi request tới Pod với địa chỉ http://hello-kube

Giờ ta tạo thử NodePort service. Tạo một file tên là hello-nodeport.yaml

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: hello-rs
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-kube
  template:
    metadata:
      labels:
        app: hello-kube
    spec:
      containers:
      - image: 080196/hello-kube
        name: hello-kube
        ports:
          - containerPort: 3000

---
apiVersion: v1
kind: Service
metadata:
  name: hello
spec:
  selector:
    app: hello-kube
  type: NodePort
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 31000
```

`kubectl apply -f hello-nodeport.yaml`

Test gửi request tới Pod với địa chỉ http://<your_worker_node_ip>:<node_port>

<img src="/Note/img/nodeport1.png">

<img src="/Note/img/nodeport2.png">

curl <your_worker_node_ip>:<node_port>


LoadBalancer:

Khi bạn chạy kubernetes trên cloud, nó sẽ hỗ trợ LoadBalancer Service, nếu bạn chạy trên môi trường không có hỗ trợ LoadBalancer thì bạn không thể tạo được loại Service này. Khi bạn tạo LoadBalancer Service, nó sẽ tạo ra cho chúng ta một public IP, mà client có thể truy cập Pod bên trong Cluster bằng địa chỉ public IP này. Ví dụ

```
apiVersion: v1
kind: Service
metadata:
  name: kubia-loadbalancer
spec:
  selector:
    app: kubia
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080
```
  
<img src="/Note/img/load.png">	 
<img src="/Note/img/load1.png">	


Ingress resource:


Ingress là một resource cho phép chúng ta expose HTTP and HTTPS routes từ bên ngoài cluster tới service bên trong cluster của chúng ta. Ingress sẽ giúp chúng ta gán một domain thực tế với service bên trong cluster. Ví dụ:

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubia
spec:
  rules:
    - host: kubia.example.com # domain name
  http:
    paths:
      - path: /
        backend:
          serviceName: kubia-nodeport # name of the service inside cluster
          servicePort: 80
```

<img src="/Note/img/load2.png">	
Ta cũng có thể mapping nhiều service với cùng một domain. Ví dụ:

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubia
spec:
  rules:
    - host: kubia.example.com
  http:
    paths:
      - path: /bar
        backend:
          serviceName: kubia-bar
          servicePort: 80
      - path: /foo
        backend:
          serviceName: kubia-foo
          servicePort: 80
```

Ingress là resource hữu dụng nhất để expose HTTP và HTTPS của Pod ra ngoài
		  

### Deployment

Recreate and RollingUpdate

Recreate

Ở cách deploy này, đầu tiên là sẽ xóa toàn bộ phiên bản (version) cũ của ứng dụng trước, sau đó ta sẽ deploy một version mới lên. Đối với kubernes thì đầu tiên ta sẽ cập nhật Pod template của ReplicaSet, sau đó ta xóa toàn bộ Pod hiện tại, để ReplicaSet tạo ra Pod với image mới.

<img src="/Note/img/dep.png">	

Với cách deploy này thì quá trình deploy quá dễ dàng, nhưng ta sẽ gặp một vấn đề rất lớn, đó là ứng dụng của chúng ta sẽ downtime với client, client không thể request tới ứng dụng của ta trong quá trình version mới được deploy lên.

Với những hệ thống ít client thì dù bạn downtime 1 phút 2 phút hoặc 1 tiếng cũng không ảnh hưởng nhiều lắm, nhưng với những hệ thống với số lượng request lớn tầm 1000-3000 request trên giây, đặt biệt là với hệ thống ngân hàng thì quá trình downtime dù chỉ 1s thì cũng không được. Nên mới sinh ra cách deploy thứ hai là RollingUpdate

RollingUpdate

Ở cách này, ta sẽ deploy từng version mới của ứng dụng lên, chắc chắn rằng nó đã chạy, ta dẫn request tới version mới của ứng dụng này, lặp lại quá trình này cho tới khi toàn bộ version mới của ứng dụng được deploy và version cũ đã bị xóa. Đối với kubernetes, ta sẽ lần lượt xóa từng Pod và ReplicaSet sẽ tạo Pod mới cho ta.

<img src="/Note/img/dep1.png">

Thì ở cách deploy này, điểm mạnh là ta sẽ giảm thời gian downtime của ứng dụng đối với client, còn điểm yếu là ta sẽ có version mới và version cũ của ứng dụng chạy chung một lúc. Và khó nhất là ta phải viết script để thực hiện quá trình này, test script này chạy đúng không, rất mất thời gian và công sức. Với các hệ thống lớn thì để viết một cái script test được quá trình deploy này thì không dễ chút nào.

Lùi lại phiên bản trước của ứng dụng

Thì ở đây chúng ta sẽ có cách là fix bug, build image mới, rồi thực hiện lại cách depoy là Recreate hoặc RollingUpdate đối với những lỗi không cần gấp lắm và không lớn lắm. Còn đối với lỗi bự và ảnh hưởng nhiều tới client thì ta thường sẽ revert lại code trên git, sau đó ta build image, thực hiện deploy lại.

Đối với kubernetes, chúng ta dùng container chạy ứng dụng thì chỉ cần update lại version của image trong Pod template bằng image trước đó, rồi dùng cách Recreate hoặc RollingUpdate để deploy lại Pod là xong, không cần revert code hoặc fixbug ngay lập tức. Dù là cách nào thì cũng sẽ tốn nhiều thời gian để thực hiện và chạy CI/CD lại.

Để giải quyết những vấn đề trên thì kubernetes cung cấp một resource là Deployment.

Deployment là gì?

Deployment là một resource của kubernetes giúp ta trong việc cập cập một version mới của úng dụng một cách dễ dàng, nó cung cấp sẵn 2 strategy để deploy là Recreate và RollingUpdate, tất cả đều được thực hiện tự động bên dưới, và các version được deploy sẽ có một histroy ở đằng sau, ta có thể rollback and rollout giữa các phiên bản bất cứ lúc nào mà không cần chạy lại CI/CD.

Khi ta tạo một Deployment, nó sẽ tạo ra một ReplicaSet bên dưới, và ReplicaSet sẽ tạo Pod. Luồn như sau Deployment tạo và quản lý ReplicaSet -> ReplicaSet tạo và quản lý Pod -> Pod run container.

<img src="/Note/img/dep2.png">

Giờ ta tạo thử Deployment nào. File config của Deployment thì đa phần cũng giống như của ReplicaSet, chỉ cần thay đổi kind thành Deployment là được. Tạo một file tên là hello-deploy.yaml:

```
apiVersion: apps/v1
kind: Deployment # change here
metadata:
  name: hello-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
      - image: 080196/hello-app:v1
        name: hello-app
        ports:
          - containerPort: 3000

---
apiVersion: v1
kind: Service
metadata:
  name: hello-app
spec:
  type: NodePort
  selector:
    app: hello-app
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 31000
```
	


Code của image 080196/hello-app:v1.

```
const http = require("http");

const server = http.createServer((req, res) => {
  res.end("Hello application v1\n")
});

server.listen(3000, () => {
  console.log("Server listen on port 3000")
})

```
`kubectl apply -f hello-deploy.yaml --record`

`192.168.30.131:31000`

<img src="/Note/img/dep3.png">

Vậy là deployment của chúng ta đã chạy thành công. Bây giờ ta sẽ tiến hành update lại ứng dụng đang chạy trong Pod. Ứng dụng của Pod mới sẽ có image là 080196/hello-app:v2. Code của image như sau:


```
const http = require("http");

const server = http.createServer((req, res) => {
  res.end("Hello application v2\n") // change v1 to v2
});

server.listen(3000, () => {
  console.log("Server listen on port 3000")
})
```
Để update lại ứng dụng trong Pod với Deployment. Ta chạy câu lệnh sau:

`kubectl set image deployment hello-app hello-app=080196/hello-app:v2`

Cấu trúc câu lệnh `kubectl set image deployment <deployment-name> <container-name>=<new-image>`


Kiểm tra qua trình update đã xong chưa:

`kubectl rollout status deploy hello-app`


Nếu câu lệnh rollout status in ra successfully, là ứng dụng của chúng ta đã cập nhật lên version mới. Ta test bằng cách gửi request tới ứng dụng ở địa chỉ localhost:31000.

<img src="/Note/img/dep4.png">


Nếu ở đây đã in ra được Hello application v2 thì ứng dụng của chúng ta đã cập nhật được version mới. Như các bạn thấy thì quá trình deploy một version của ứng dụng cực kì đơn giản nếu chúng ta xài Deployment. Ta cũng có thể chọn deploy strategy ta muốn cực kì đơn giản bằng cách chỉ định trong thuộc tính strategy của Deployment.

```
apiVersion: apps/v1
kind: Deployment 
metadata:
  name: hello-app
spec:
  replicas: 3
  strategy: # change here
    type: RollingUpdate # strategy type is Recreate or RollingUpdate. Default is RollingUpdate
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
      - image: 080196/hello-app:v1
        name: hello-app
        ports:
          - containerPort: 3000
```

Cách Deploymnet update Pod:

Thì ở dưới kubernetes, khi ta thay đổi image của Deployment, nó sẽ tạo ra một ReplicaSet khác, và ReplicaSet đó sẽ giữ template Pod mới, và các Pod mới sẽ được tạo ra bởi ReplicaSet.

<img src="/Note/img/dep5.png">

Và ReplicaSet cũ sẽ không bị xóa, mà trong trường này thuộc tính replicas của nó sẽ được cập nhật lại là 0. Vì sao ReplicaSet cũ không bị xóa thì nó sẽ có vai trò là ở trong quá trình mà ta update version mới của ứng dụng, và ta phát hiện version mới nó bị lỗi, ta muốn rollback lại version trước, thì ReplicaSet cũ của ta vẫn ở đó, ta sẽ rollback lại dễ dàng hơn.

Rollback lại version trước khi version mới của ứng dụng bị lỗi
Image của version mới của chúng ta là 080196/hello-app:v3. Code của image như sau, khi ta gọi request tới nó hơn 3 lần thì nó sẽ trả về lỗi:

```
const http = require("http");

let requestCount = 0;

const server = http.createServer((req, res) => {
  if (++requestCount > 3) {
    res.writeHead(500); // set status code 500
    res.end("Application v3 have some internal error has occurred!\n");
    return;
  }

  res.end("Hello application v3\n");
});

server.listen(3000, () => {
  console.log("Server listen on port 3000")
})
```


Cập nhật lại Deployment

`kubectl set image deployment hello-app hello-app=080196/hello-app:v3`

<img src="/Note/img/dep6.png">

Như bạn thấy ở đây là ứng dụng v3 của chúng ta đã bị lỗi, và ta muốn nhanh chóng rollback lại version trước đó là v2, để không ảnh hưởng tới client. Để làm được việc đó thì là dùng câu lệnh kubectl rollout. Trước tiên bạn có thể kiểm tra lịch sử các lần ứng dụng của chúng ta đã cập nhật.

`kubectl rollout history deploy hello-app`

Ta sẽ thấy là có 3 lần chúng ta đã cập nhập ứng dụng. Ta đang ở revision là 3, ta muốn quay lại revision là 2, lúc ứng dụng của ta chưa có lỗi

`kubectl rollout undo deployment hello-app --to-revision=2`

<img src="/Note/img/dep7.png">

Ứng dụng của ta đã in ra Hello application v2, vậy là ta đã rollback về version trước thành công. Giờ ta chỉ việc fixbug và deploy version mới không có bug. Mọi chuyện trở nên dễ dàng hơn, không cần phải revert code trên git hay gì cả, vẫn có thời gian fixbug mà không ảnh hưởng tởi client. Như các bạn thấy đây là điểm mạnh của việc ta sử dụng Deployment để chạy ứng dụng của chúng ta. Đây là minh họa của revision history.

<img src="/Note/img/dep8.png">

Về revision thì mặc định nó sẽ lưu là 10, bạn có thể điều chỉnh số lượng history bạn muốn lưu lại bằng thuộc tính revisionHistoryLimit của Deployment

```
apiVersion: apps/v1
kind: Deployment 
metadata:
  name: hello-app
spec:
  revisionHistoryLimit: 1
  replicas: 3
  ...
```

Ngoài ra Deployment còn có một điểm mạnh rất tuyệt vời nữa là có thể giúp ta config để deploy ứng dụng zero downtime, tuy ta dùng RollingUpdate nhưng không cũng không chắc được là ứng dụng của chúng ta zero downtime. Về phần config cái này thì mình sẽ nói ở các bài sau. Xóa resource đi nhé:

Dưới đây là config CI/CD dùng Deployment với gitlabCI trong thực tế, các bạn có thể tham khảo thêm:

```
stages:
  - build
  - deploy

image:
  name: docker:stable
  
variables:
  DOCKER_HOST: tcp://docker:2375
  DOCKER_TLS_CERTDIR: ""

services:
  - docker:stable-dind

build root image:
  stage: build
  before_script:
    - echo "$REGISTRY_PASSWORD" | docker login $CI_REGISTRY -u $REGISTRY_USER --password-stdin
  script:
    - docker pull $CI_REGISTRY_IMAGE:latest || true
    - docker build . --cache-from $CI_REGISTRY_IMAGE:latest -t $CI_REGISTRY_IMAGE:latest -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  only:
    - main

deploy k8s:
  stage: deploy
  image: bitnami/kubectl
  script:
    - |
      kubectl -n testing set image deployment microservice microservice=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  when: manual
  only:
    - main

```	
 
### Volume: gắn disk storage vào container:

Volume hiểu đơn giản chỉ là một mount point từ hệ thống file của server vào bên trong container.

Tại sao ta cần volume thì đối với container, những thứ ta ghi vào filesystem của nó thì chỉ tồn tại khi container còn chạy. Khi một thằng Pod bị xóa và tạo lại, container mới sẽ được tao ra, lúc này thì những thứ ta ghi ở container trước sẽ bị mất đi. Nếu ta muốn giữ lại những dữ liệu đó thì ta phải sử dụng volume.

Trong kubernetes thì sẽ có những loại volume như sau:
```
- EmptyDir
- hostPath
- gitRepo
- nfs
- gcePersistentDisk, awsElasticBlockStore, azureDisk (cloud storage)
- cinder, cephfs, iscsi, flocker, glusterfs, quobyte, rbd, flexVolume, vsphereVolume, photonPersistentDisk, scaleIO
- configMap, secret, downwardAPI
- PersistentVolumeClaim
```

Những loại volume trên được phân chia thành 3 dạng chính:

- Volume dùng để chia sẻ dữ liệu giữa các container trong Pod
- Volume đính kèm vào trong filesystem một node
- Volume đính kèm vào cluster và các node khác nhau có thể truy cập

Ta chỉ cần nhớ một vài loại hay sử dụng nhất là emptyDir, hostPath, cloud storage, PersistentVolumeClaim. Các loại secret, downwardAPI, configMap ta sẽ nói ở các bài tiếp.

### Sử dụng emptyDir volume để share data giữa các containers 
 
EmptyDir là loại volume đơn giản nhất, nó sẽ tạo ra một empty directory bên trong Pod, các container trong một Pod có thể ghi dữ liệu vào bên trong nó. Volume chỉ tồn tại trong một lifecycle của Pod, dữ liệu trong loại volume này chỉ được lưu trữ tạm thời và sẽ mất đi khi Pod bị xóa. Ta dùng loại volume này khi ta chỉ muốn các container có thể chia sẻ dữ liệu lẫn nhau và không cần lưu trữ dữ liệu lại. Ví dụ là dữ liệu log từ một thằng container chạy web API, và ta có một thằng container khác sẽ truy cập vào log đó để xử lý log.

Ta sẽ làm một ví dụ đơn giản cho dễ hiểu hơn, tạo một file emptydir.yaml với config như sau:

```
apiVersion: v1
kind: Pod
metadata:
  name: fortune
spec:
  containers:
    - name: html-generator
      image: luksa/fortune
      volumeMounts:
        - name: html # The volume called html is mounted at /var/htdocs in the container
          mountPath: /var/htdocs
    - name: web-server
      image: nginx:alpine
      ports:
        - containerPort: 80
          protocol: TCP
      volumeMounts:
        - name: html # The volume called html is mounted at /usr/share/nginx/html in the container
          mountPath: /usr/share/nginx/html
          readOnly: true
  volumes: # define volumes
    - name: html # name of the volumes
      emptyDir: {} # define type is emptyDir
```
 
Code của script trong container luksa/fortune.

```
#!/bin/bash
trap "exit" SIGINT
mkdir /var/htdocs

while :
do
  echo $(date) Writing fortune to /var/htdocs/index.html
  /usr/games/fortune > /var/htdocs/index.html
  sleep 10
done
``` 
 
Container html-generator này sẽ cứ mỗi 10 giây sẽ tạo ra một nội dung bất kì và lưu nó vào file index.html. Và ta sẽ có một container khác, tên là web-server, sẽ start một server và hosting nội dung ở folder /usr/share/nginx/html (folder mặc định của nginx).

Ở đây ta có một emptyDir volume tên là html, được mount vào container html-generator ở folder /var/htdocs và container html-generator sẽ tạo một file html index.html ở trong emptyDir volume này. Và emptyDir volume này được mount tới container web-server, ở folder /usr/share/nginx/html. Nên khi ta truy cập container web thì ta sẽ thấy được những nội dung mà container html-generator đã tạo ra.

<img src="/Note/img/dep9.png">

Test xem nó có hoạt động đúng không.
```
kubectl apply -f emptydir.yaml
kubectl port-forward fortune 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80


curl http://localhost:8080
```
Lưu ý là chỉ dùng emptyDir để chia sẻ dữ liệu giữa những container chứ không dùng để lưu persistent data. Tiếp theo là một loại volume rất hữu ích cho các static website là gitRepo. 
 

### Sử dụng gitRepo để clone git repository vào container

<img src="/Note/img/vol.png">

gitRepo là loại volume cũng giống emptyDir là sẽ tạo một empty folder, và sau đó nó sẽ clone code của git repository vào folder này.

```
apiVersion: v1
kind: Pod
metadata:
  name: gitrepo-volume-pod
spec:
  containers:
    - image: nginx:alpine
      name: web-server
      volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
          readOnly: true
      ports:
        - containerPort: 80
          protocol: TCP
  volumes:
    - name: html
      gitRepo: # gitRepo volume
        repository: https://github.com/luksa/kubia-website-example.git # The volume will clone this Git repository
        revision: master # master branch
        directory: . # cloned into the root dir of the volume.
```


Ở ví dụ này thì html volume được tạo ra với empty folder, sau đó code ở kubia-website-example repository sẽ được clone xuống ở root folder vì ta chỉ định . (dot), volume này được mount vào container web-server ở folder /usr/share/nginx/html. Khi ta truy cập vào Pod, ta sẽ thấy được nội dung static file của kubia-website-example git repository.

Đối với gitRepo volume thì hiện tại không có hỗ trợ private repository, muốn clone được private repository thì ta phải sử dụng một pattern gọi là sidecar containers. Mình sẽ viết một series về kubernetes pattern đển nói về các pattern trong kubernetes, sidecar containers sẽ được mình nói trong series đó sau.

2 loại volume ta vừa nói chỉ được dùng để share data giữ các container, nhưng trong thực tế thì ta lại có nhu cầu rất lớn về việc lưu trữ persistent data, ví dụ như là Pod chạy container database, ta đâu thể để Pod của chúng ta xóa đi là dữ liệu lưu trữ của ta bị mất được, nên kubernetes có cung cấp cho ta các loại volume để lưu trữ persistent data, đầu tiên ta sẽ nói về hostPath volume.

Sử dụng hostPath để truy cập filesystem của worker node:

<img src="/Note/img/vol1.png">

hostPath là loại volume sẽ tạo một mount point từ Pod ra ngoài filesystem của node. Đây là loại volume đầu tiên ta nói mà có thể dùng để lưu trữ persistent data. Dữ liệu lưu trong volume này chỉ tồn tại trên một worker node và sẽ không bị xóa đi khi Pod bị xóa.

Ví dụ như sau:

```
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-volume
spec:
  containers:
    - image: nginx:alpine
      name: web-server
      volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
          readOnly: true
        - name: log # log volume
          mountPath: /var/log/nginx # mounted at /var/log/nginx in the container
      ports:
        - containerPort: 80
          protocol: TCP
  volumes:
    - name: html
      gitRepo: # gitRepo volume
        repository: https://github.com/luksa/kubia-website-example.git # The volume will clone this Git repository
        revision: master # master branch
        directory: . # cloned into the root dir of the volume.
    - name: log
      hostPath: # hostPath volume
        path: /var/log # folder of woker node
```

Ở đây ta sẽ xài lại ví dụ ở trên, và thêm vào một volume tên là log, sẽ được mount tới file system của worker node ở folder /var/log, và volume này sẽ được mount tới container web-server ở folder /var/log/nginx. Lúc này thì tất cả log của container sẽ được lưu trữ ở folder /var/log của worker node.

<img src="/Note/img/vol2.png">

Đối với loại volume này thì Pod của ta cần phải được tạo đúng worker node thì ta mới có được dữ liệu trước đó, nếu Pod của ta được tạo ở một worker node khác thì khi đó Pod sẽ không có dữ liệu cũ, do dữ liệu vẫn nằm ở worker node cũ. Loại volume này ta không sử dụng nó cho việc lưu trữ persistent data hoàn toàn được. Cái ta muốn là dù Pod được tạo ở worker node nào thì dữ liệu của ta vẫn có, để mount được vào trong container.

### Sử dụng cloud storage để lưu trữ persistent data

Loại volume này chỉ được hỗ trợ trên các nền tảng cloud, giúp ta lưu trữ persistent data, kể cả khi Pod được tạo ở các worker node khác nhau, dữ liệu của ta vẫn tồn tại cho container. 3 nền tảng cloud mà phổ biến nhất là AWS, Goolge Cloud, Azure tương ứng với 3 loại volume là gcePersistentDisk, awsElasticBlockStore, azureDisk.

Thì ở ví dụ này ta sẽ xài Google Cloud, nếu các bạn có tài khoản Google Cloud và có kiến thức về nó rồi thì nên làm theo, còn không thì ta chỉ xem ví dụ cho biết thêm. Ta dùng câu lệnh sau để tạo một Persistent Disk trên google cloud:

gcloud compute disks create --size=1GiB --zone=europe-west1-b mongodb

Tạo một file tên là gcepd.yaml với config như sau:

```
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
spec:
  containers:
    - image: mongo
      name: mongodb
      ports:
        - containerPort: 27017
          protocol: TCP
      volumeMounts:
        - name: mongodb-data
          mountPath: /data/db
  volumes:
    - name: mongodb-data
      gcePersistentDisk: # google cloud disk volume
        pdName: mongodb # name of the persistent disk on google cloud
        fsType: ext4
```


Vì loại volume này ta sẽ sử dụng GCE persistent disk, nên nó không thuộc về bất cứ worker node nào mà sẽ nằm riêng lẻ một mình nó. Khi Pod của chúng ta dù được tạo ở bất kì worker node nào thì ta vẫn có thể mount được tới volume này. Và dữ liệu của volume này vẫn được giữ nguyên khi Pod xóa đi.

Để sử dụng volume của cloud storage khác thì ta chỉ cần thay đổi loại volume là được, rất đơn giản, ví dụ như sau:

<img src="/Note/img/vol3.png">

```
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
spec:
  ...
  volumes:
    - name: mongodb-data
      awsElasticBlockStore: # using AWS ElasticBlockStore instead of gcePersistentDisk
        pdName: aws-ebs # name of the EBS on AWS
        fsType: ext4
```

### PersistentVolumeClaims: tách Pod ra khỏi kiến trúc storage bên dưới

PersistentVolumeClaims, PersistentVolumes

Với PersistentVolumes là resource sẽ tương tác với kiến trúc storage bên dưới, và PersistentVolumeClaims sẽ request storage từ PersistentVolumes, tương tự như Pod. Pods tiêu thụ node resources và PersistentVolumeClaims tiêu thụ PersistentVolumes resources.


Thông thường khi làm việc với kubernetes ta sẽ có 2 role là:

- kubernetes administrator: Người dựng và quản lý kubernetes cluster, cài những plugin và addons cần thiết cho kubernetes cluster.
- kubernetes developer: Người mà sẽ viết file config yaml để deploy ứng dụng lên trên kubernetes.

Một kubernetes administrator sẽ setup kiến trúc storage bên dưới và tạo PersistentVolumes để cho kubernetes developer request và xài.

<img src="/Note/img/vol4.png">

### Tạo PersistentVolumes

Bây giờ ta sẽ là một cluster administrator và ta cần tạo PersistentVolume để cho cluster developer có thể request và xài. Tạo một file tên pv-gcepd.yaml với config như sau:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv
spec:
  capacity:
    storage: 10Gi # size of the storage
  accessModes: # access mode
    - ReadWriteOnce # can be mounted by a single wokrer node for reading and writing
    - ReadOnlyMany # can be mounted by a multiple wokrer node for reading only
  persistentVolumeReclaimPolicy: Retain
  gcePersistentDisk:
    pdName: mongodb
    fsType: ext4
```

Khi cluster administrator tạo một PV, ta cần chỉ định size của PV này là bao nhiêu, và cần chỉ định access modes của nó, có thể được đọc và ghi bởi một node hay nhiều node. Ở ví dụ trên, thì chỉ có 1 node có quyền ghi vào PV này, còn nhiều node khác có quyền đọc từ PV này, kiến trúc storage của PV này xài là gcePersistentDisk, ta đã nói về loại volume này ở bài trước. Ta tạo thử PV ta vừa mới viết là list nó ra xem thử.


Khi cluster administrator tạo một PV, ta cần chỉ định size của PV này là bao nhiêu, và cần chỉ định access modes của nó, có thể được đọc và ghi bởi một node hay nhiều node. Ở ví dụ trên, thì chỉ có 1 node có quyền ghi vào PV này, còn nhiều node khác có quyền đọc từ PV này, kiến trúc storage của PV này xài là gcePersistentDisk, ta đã nói về loại volume này ở bài trước. Ta tạo thử PV ta vừa mới viết là list nó ra xem thử


```
kubectl apply -f pv-gcepd.yaml
kubectl get pv
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
mongodb-pv   10Gi       RWO,ROX        Retain           Available                                   46s
```
Bây giờ thì ta đã có PV. Tiếp theo thì ta sẽ đóng vai cluster developer để tạo PVCs và sẽ tiêu thụ PV này.

```
PersistentVolumes sẽ không thuộc về bất kì namespace nào. Trong kubernetes thì có tồn tại 2 loại resource là namespace resource và cluster resource. Như Pod, Deployment thì là namspace resource, khi tạo Pod ta có thể chỉ định thuộc tính namespace và Pod sẽ thuộc về namespace đó, nếu ta không chỉ định namespace thì Pod sẽ thuộc default namespace. Còn cluster resource sẽ không thuộc về namespace nào, như là Node, PersistentVolumes resource.
```

<img src="/Note/img/vol5.png">
	
- Tạo PersistentVolumeClaim tiêu thụ PersistentVolumes

Bây giờ ta là developer, ta cần deploy Pod mà cần xài volume để lưu trữ persistent data. Tạo một file tên là mongodb-pvc.yaml với config như sau:

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
spec:
  resources:
    requests:
      storage: 10Gi # request 10Gi storage
  accessModes:
    - ReadWriteOnce # only allow one node can be read and write
  storageClassName: ""
```

Ở đây ta sẽ tạo một PVC tên là mongodb-pvc mà request 10Gi storage, nếu có PV nào đáp ứng được nó, thì PVCs này sẽ tiêu thụ storage của PV đó. Và thằng PV này chỉ cho phép một thằng worker node có quyền đọc và ghi vào nó. Ở đây có một thuộc tính storageClassName ta chỉ định là rỗng, ta sẽ nói về thuộc tính này ở phần dưới. Tạo PVCs và list nó ra xem thử.

```
root@k8s1:/opt/release/20220626# kubectl apply -f mongodb-pvc.yaml
persistentvolumeclaim/mongodb-pvc created
root@k8s1:/opt/release/20220626# kubectl get pvc
NAME          STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mongodb-pvc   Bound    mongodb-pv   10Gi       RWO,ROX                       8s
```

PVCs của ta đã được tạo ra và đã được Bound vào trong PVC, list PV ta tạo ở trên ra xem thử nó đã được xài bởi PVCs chưa.

```
root@k8s1:/opt/release/20220626# kubectl apply -f pv-gcepd.yaml
persistentvolume/mongodb-pv unchanged
root@k8s1:/opt/release/20220626# kubectl get pv
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
mongodb-pv   10Gi       RWO,ROX        Retain           Bound    default/mongodb-pvc                           5m40s
```

Ta thấy status của PV đã chuyển thành Bound và cột CLAIM đã hiển thị PVCs đang tiêu thụ nó.


Tạo Pod sử dụng PersistentVolumeClaim

Bây giờ ta sẽ tạo Pod xài PVCs, tạo một file tên là mongodb-pod-pvc.yaml với config như sau:

```
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
spec:
  containers:
    - image: mongo
      name: mongodb
      volumeMounts:
        - name: mongodb-data
          mountPath: /data/db
      ports:
        - containerPort: 27017
          protocol: TCP
  volumes:
    - name: mongodb-data
      persistentVolumeClaim:
        claimName: mongodb-pvc # specify PVCs we want to use
```

kubectl create -f mongodb-pod-pvc.yaml

Như chúng ta muốn, dữ liệu ta tạo ở bài trước vẫn nằm ở đây, nằm trong Persistent Disk của google cloud.

<img src="/Note/img/vol6.png">

Lợi ích của việc xài PersistentVolumeClaim

Trong bài viết này thì các bạn sẽ thấy so sánh với volume thì xài PersistentVolumeClaim ta cần làm nhiều bước hơn. Nhưng với góc nhìn của developer khi làm thực tế thì bây giờ ta chỉ cần tạo PVCs và chỉ định size của nó, sau đó trong Pod ta chỉ cần chỉ định tên của PVCs, ta không cần phải làm việc với kiến trúc storage bên dưới node của ta, và ta cũng chả cần biết là dữ liệu ta được lưu ở worker node hay là ở storage của cloud hay là ở chỗ khác. Những thứ đó là việc của cluster administrator.

Và file config của PVCs ta có thể xài lại ở những cluster khác được, trong khi ta xài volume thì ta cần phải xem là cluster đó hỗ trợ những kiến trúc storage nào trước, nên một file config có thể khó xài được ở những cluster khác nhau.

### Recycling PersistentVolumes

Khi tạo PV thì ta để ý có một thuộc tính là persistentVolumeReclaimPolicy, thuộc tính này sẽ định nghĩa hành động của PV như thế nào khi PVCs bị xóa đi, có 3 mode là:

- Retain
- Recycle
- Delete
Ở mode Retain policy, khi ta delete PVCs thì PV của ta vẫn tồn tại ở đó, nhưng nó PV sẽ ở trạng thái là Release chứ không phải Available như ban đầu, vì nó đã được sử dụng bởi PVCs và chứa dữ liệu rồi, nếu để thằng PVCs bound vào thì có thể gây ra lỗi, dùng mode này khi ta muốn lỡ có ta có xóa PVCs thì dữ liệu của ta vẫn con đó, việc ta cần làm là xóa PV bằng tay, tạo ra PV mới là tạo ra PVCs mới để bound vào lại.


Ta xóa thử thằng PVCs.

```
$ kubectl delete pod mongodb
pod "mongodb" deleted
$ kubectl delete pvc mongodb-pvc
persistentvolumeclaim "mongodb-pvc" deleted
$ kubectl get pv
NAME        CAPACITY  ACCESSMODES  STATUS    CLAIM              
mongodb-pv  10Gi      RWO,ROX      Released  default/mongodb-pvc
```

Ở mode Recycle policy, khi ta delete PVCs thì PV của ta vẫn tồn tại ở đó, nhưng lúc này dữ liệu trong PV sẽ bị xóa đi luôn và trạng thái sẽ là Available để cho một thằng PVCs khác có thể tiêu thụ nó. Hiện tại thì GCE Persistent Disks không có hỗ Recycle policy.

Ở mode Delete policy, khi ta xóa PVCs đi thì PV ta cũng bị xóa theo luôn.

Ta có thể thay đổi policy của một thằng PV đang có, như là chuyển từ Delete sang Retain để tránh mất dữ liệu.

<img src="/Note/img/vol7.png">

### Tự động cấp PersistentVolumes (Dynamic provisioning)

Ta đã thấy cách sử dụng PV và PVCs với nhau để developer không cần làm việc với kiến trúc storage bên dưới. Nhưng ta vẫn cần một administrator setup những thứ đó trước. Để tránh việc đó thì Kubernetes có cung cấp ta một cách để tự động tạo PV bên dưới.

Cách mà administrator setup trước gọi là Pre provisioning, còn cách tự động gọi là Dynamic provisioning.

Để làm được việc này thì ta sẽ sử dụng StorageClasses với một provisioner (được cloud hỗ trợ mặc định), còn các môi trường không phải cloud thì ta phải cài provisioner.

Tạo StorageClass

Đây là resource sẽ tự động tạo PV cho ta, ta chỉ cần tạo StorageClass một lần, thay vì phải config và tạo một đống PV. Tạo một file tên là storageclass-fast-gcepd.yaml với config như sau:

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd # use gce-pd provisioner
parameters:
  type: pd-ssd
  zone: europe-west1-b
```

Ở đây ta tạo một StorageClass tên là fast, sử dụng gce-pd provisioner mà sẽ giúp ta tự động tạo PV bên dưới. Khi ta tạo một PVCs, ta sẽ chỉ định storageClassName là fast. Lúc này thì PVCs ta sẽ request tới StorageClass, và StorageClass sẽ tự động tạo một thằng PV bên dưới cho PVCs xài.

Tạo một file tên là mongodb-pvc-dp.yaml với config như sau:

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
spec:
  storageClassName: fast # This PVC use fast StorageClass
  resources:
    requests:
      storage: 100Mi
  accessModes:
    - ReadWriteOnce
```

kubectl apply -f mongodb-pvc-dp.yaml

```
kubectl get pvc mongodb-pvc
NAME         STATUS  VOLUME        CAPACITY  ACCESSMODES  STORAGECLASS
mongodb-pvc  Bound   pvc-1e6bc048  1Gi       RWO          fast
$ kubectl get pv
NAME          CAPACITY  ACCESSMODES RECLAIMPOLICY  STATUS     STORAGECLASS
mongodb-pv    10Gi       RWO,ROX     Retain         Released
pvc-1e6bc048  1Gi        RWO         Delete         Bound     fast
```

Ta sẽ thấy có một thằng pvc-1e6bc048 tự động được StorageClass tạo ra.

<img src="/Note/img/vol8.png">

Dynamic provisioning mà không cần chỉ định storage class

Khi ta không chỉ định thuộc tính storageClassName trong PVCs thì nó sẽ xài storage class mặc định. Ta list thử storage class ra. Đây là storage class ở trên google cloud, ở dưới local của bạn có thể sẽ khác: 
 
```
kubectl get sc
NAME                TYPE
fast                kubernetes.io/gce-pd
standard (default)  kubernetes.io/gce-pd

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
spec:
  resources:
    requests:
      storage: 100Mi
  accessModes:
    - ReadWriteOnce

```

Như ở file config trên, ta không chỉ định thuộc tính storageClassName, nên PVCs này sẽ mặc định xài standard storageClassName.

Còn nhở ở file config ban đầu của chúng ta, ta chỉ định thuộc tính storageClassName = "" chứ?

```
...
kind: PersistentVolumeClaim
spec:
    storageClassName: ""
...
```

Ý nghĩa của nó ở đây là ta sẽ không sử dụng storageClassName để tự động tạo PV mà ta sẽ xài PV có sẵn. Tại sao lại có như vậy, sao không xài storage class luôn đi cho nhanh mà phải tạo PV rồi xài, mất công thế? Như đã nói ở trên ta muốn xài được storage class thì ta cần provisioner, thì hiện tại chỉ có cloud là hỗ trợ sẵn provisioner thôi, nếu bạn cài trên data center thông thường thì không có provisioner sẵn cho bạn để bạn xài chức năng dynamic provisioning được.


### ConfigMap and Secret: truyền cấu hình vào container

Chỉ định biến môi trường cho container
Thì cấu hình của ứng dụng trong một container thường được truyền thông qua biến môi trường (environment variables - viết tắt env).

<img src="/Note/img/c1.png">

Kubernetes có cung cấp cho chúng ta cách truyền một list env vào bên trong từng container của Pod. Ví dụ, ta có một image 080196/hello-env với code như sau:

```
const http = require("http");

const server = http.createServer((req, res) => {
  console.log("Hello env\n")
});

server.listen(process.env.PORT, () => {
  console.log("Server listen on port ", process.env.PORT)
})
```
	
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
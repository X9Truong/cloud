### Setting Up a Kubernetes Cluster on Ubuntu 18.04

* Contents


### Configure the Master Node

- Install Dependencies

Install and configure prerequisites:

`swapoff -a`

```
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

```
apt-get update && apt-get install -y apt-transport-https
curl -s https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
apt update && apt install -qy docker-ce
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" \
  > /etc/apt/sources.list.d/kubernetes.list
apt-get update && apt-get install -y kubeadm kubelet kubectl
```

```
sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
"exec-opts": ["native.cgroupdriver=systemd"],
"log-driver": "json-file",
"log-opts": {
"max-size": "100m"
},
"storage-driver": "overlay2"
}
EOF
```

```
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl status kubelet
```

Configure Kubernetes
Init Kubernetes:


`kubeadm init --apiserver-advertise-address=<MASTER_IP> --pod-network-cidr=10.244.0.0/16`

Take note of the join token command. For example:

```
kubeadm join 192.168.30.131:6443 --token 0s2wfs.loq1y0dcsrnckty4         --discovery-token-ca-cert-hash sha256:c73a8db1a1be2f737d70184ca7f7acfed24fa71b75449102d32257a5620b80a9
```

Set up the Kubernetes config as the user you created:

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Again, as the new user, deploy a flannel network to the cluster:

`kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`

### Configure the Worker Nodes

Install Dependencies

```
apt-get update && apt-get install -y apt-transport-https
curl -s https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
apt update && apt install -qy docker-ce
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" \
  > /etc/apt/sources.list.d/kubernetes.list
apt-get update && apt-get install -y kubeadm kubelet kubectl
```

```
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl status kubelet
```

Configure Kubernetes

`kubeadm join 192.168.30.131:6443 --token 0s2wfs.loq1y0dcsrnckty4         --discovery-token-ca-cert-hash sha256:c73a8db1a1be2f737d70184ca7f7acfed24fa71b75449102d32257a5620b80a9`


### Lab

https://github.com/cloudnativedevops/demo

` docker container run -p 9999:8888 --name hello cloudnatived/demo:hello`

- Check `192.168.30.131:9999`

```
cat Dockerfile
FROM golang:1.17-alpine AS build

WORKDIR /src/
COPY main.go go.* /src/
RUN CGO_ENABLED=0 go build -o /bin/demo

FROM scratch
COPY --from=build /bin/demo /bin/demo
ENTRYPOINT ["/bin/demo"]
```

`docker login` ### use hub.docker.com

`docker image build -t helloworld .`

`docker image tag helloworld anhnt/hello:v1`

`docker image push yourid/myhello:v1`

`kubectl run demo --image=anhbka/hello:v1 --port=9999 --labels app=demo`

`kubectl port-forward pod/demo 9999:8888`

`kubectl get pods --selector app=demo`


* Deployment: Run app stateless

* DaemonSet: Run one pod on one node

* StatefulSet: App Run statefull

* Job: Run once time

* CronJob: Run multi times

`kubectl create deployment --image=anhbka/hello:v1 my-hello`

```
root@k8s1:~# kubectl get all
NAME                            READY   STATUS    RESTARTS   AGE
pod/my-hello-66994644db-79j49   1/1     Running   0          5s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   11m

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-hello   1/1     1            1           5s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/my-hello-66994644db   1         1         1       5s
root@k8s1:~# kubectl get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE     IP           NODE   NOMINATED NODE   READINESS GATES
my-hello-66994644db-79j49   1/1     Running   0          2m57s   10.244.1.4   k8s2   <none>           <none>

```

```
root@k8s1:~# kubectl get all -o wide
NAME                            READY   STATUS    RESTARTS   AGE    IP           NODE   NOMINATED NODE   READINESS GATES
pod/my-hello-66994644db-5shj4   1/1     Running   0          22s    10.244.1.5   k8s2   <none>           <none>
pod/my-hello-66994644db-79j49   1/1     Running   0          6m8s   10.244.1.4   k8s2   <none>           <none>

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   17m   <none>

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE    CONTAINERS   IMAGES            SELECTOR
deployment.apps/my-hello   2/2     2            2           6m8s   hello        anhbka/hello:v1   app=my-hello

NAME                                  DESIRED   CURRENT   READY   AGE    CONTAINERS   IMAGES            SELECTOR
replicaset.apps/my-hello-66994644db   2         2         2       6m8s   hello        anhbka/hello:v1   app=my-hello,pod-template-hash=66994644db
```

### Install helm

```
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

OR

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

```
helm list
helm delete --purge $NAME
```

* Install demo

`helm install demo ./k8s/demo`

```
vim k8s/demo/values.yaml
environment: development
container:
  name: demo
  port: 8888
  image: anhbka/hello
  tag: v1
replicas: 2
```

```
root@k8s1:~/demo/hello-helm# helm list
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
demo    default         2               2021-11-06 18:52:01.891793797 +0700 +07 deployed        demo-1.0.1
```


* Use helm install Grafana, Prometheus

```
cat custom-values.yml
grafana:
  adminPassword: 123456@
```

```
kubectl create namespace prometheus
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus -f  prometheus-community/kube-prometheus-stack --namespace prometheus
```

`kubectl --namespace prometheus get pods -l "release=prometheus"`

`kubectl get svc -n prometheus`

`kubectl edit svc prometheus-grafana -n prometheus`


```
username: admin
password: prom-operator
```

### Alias command

```
alias k='kubectl'
alias kg='kubectl get'
alias kgpo='kubectl get pods'
alias kgpoojson='kubectl get pods -o=json'
alias kgpon='kubectl get pods --namespace'
alias ksysgpooyamll='kubectl --namespace=kube-system get pods -o=yaml -l'
```

```
alias krm='kubectl delete'
alias krmf='kubectl delete -f'
alias krming='kubectl delete ingress'
alias krmingl='kubectl delete ingress -l'
alias krmingall='kubectl delete ingress --all-namespaces'
```

```
alias ka='kubectl apply -f'
alias klo='kubectl logs -f'
alias kex='kubectl exec -i -t'
```

```
root@k8s1:~# kubectl get pods -n kube-system -o json |jq '.items[].metadata.name'
"coredns-78fcd69978-mrk2c"
"coredns-78fcd69978-xwt6c"
"etcd-k8s1"
"kube-apiserver-k8s1"
"kube-controller-manager-k8s1"
"kube-flannel-ds-bz6tp"
"kube-flannel-ds-sw555"
"kube-proxy-5nm6m"
"kube-proxy-xdnh4"
"kube-scheduler-k8s1"
```

### Create file yaml

`kubectl create deployment --image=anhbka/hello:v1 my-hello --dry-run=client -o yaml > pod.yml`

### Check logs

- Stream logs use fluentd elasticsearch

```
kubectl logs $NAMESPACE -tail=20 $NAMEPOD 
kubectl logs $NAMESPACE -tail=20 $NAMEPOD -c $NAME_CON
kubectl logs $NAMESPACE --all-containers=true $NAMEPOD
```

### Access container

`kubectl exec -it $NAME_CON /bin/sh`

`kubectl exec -it -n prometheus prometheus-grafana-7cd6d569b-cqq4r sh`


### Work with contexts

```
kubectl config get-contexts
kubectl cluster-info
kubectl use-contexts gke
kubectl config set-context my-app --cluster=gke --namespace=myapp
kubectl config current-context
```

### Tains and Tolerations 


### Install Helm bitnami

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm search repo bitnami
helm install my-release bitnami/metrics-server
```


```
wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.4.1/components.yaml
kubectl edit deploy -n kube-system metrics-server
helm upgrade --namespace default my-release bitnami/metrics-server --set apiService.create=true

Add: - --kubelet-insecure-tls
imagePullPolicy: IfNotPresent
args:
  - --cert-dir=/tmp
  - --secure-port=4443
  - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
  - --kubelet-insecure-tls
```

### Install apache benchmark

```
apt install apache2-utils -y
ab -c 100 -n 500000 -t 100000 http:10.100.119.159:32040/
```
`kubectl get hpa`

### Increase load

`kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"`


### Canary vs. blue/green vs. rolling deployment

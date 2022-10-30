### How to Install and Use Argo CD

### Step 1: Prerequisites

a) You should have a Kubernetes Cluster running with atleast one control node and one worker node.

b) You should have kubectl command line tool installed.

c) You should have a valid kubeconfig file.

d) You should have sudo or root access to run privileged commands.

### Step 2: Create a Namespace

First we need to create a namespace where we will install argo CD and then deploy our apps. Here we are creating a namespace called argocd using kubectl create namespace argocd command as shown below.

`kubectl create namespace argocd`


### Step 3: Install Argo CD
In the next step we need to install Argo CD using below kubectl apply command.

`kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`

### Step 4: Change argocd-server Service Type to LoadBalancer

By default, Argo CD API Server is not exposed with an external IP so to access the Server you need to expose it by either using LoadBalancer, Ingress or Port Forwarding method. For the demo purpose we are going to use LoadBalancer method. To expose Argo CD API Server with an external IP, you need to change the argocd-server service type to LoadBalancer using below kubectl patch command.

`kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'`

### Step 5: Install argocd Utility

The next step is to install the argocd utility if it is not already installed in your System. You can download the binary under /usr/local/bin directory and provide the execute permission using below commands.

```
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
```

### Step 6: Login to Argo CD Server

To login to Argo CD Server, we are going to use default admin user account. You can get the secrets from the password field of argocd-initial-admin-secret and then store that secret in a variable called argoPass as shown below. Finally pass this variable as an argument to argocd login command for authentication.

```
argoPass=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
argocd login --insecure --grpc-web k3s_master:32761 --username admin --password $argoPass
```
### Step 7: Register a Cluster (Optional)

This step is only required when you are deploying application to an external cluster. But if you are deploying to an internal cluster then you need to use https://kubernetes.default.svc as the application's K8s API server address. To register a cluster, first list all the cluster context using kubectl config get-contexts -o name command as shown below.

`kubectl config get-contexts -o name`

Then choose a context name from the above command output and pass it to argocd cluster add command. For example, if you are choosing a context called docker-desktop then you need to run argocd cluster add docker-desktop command as shown below.

`argocd cluster add docker-desktop`

The above command will install a ServiceAccount (argocd-manager), into the kube-system namespace of that kubectl context, and binds the service account to an admin-level ClusterRole. Argo CD uses this service account token to perform its management tasks (i.e. deploy/monitoring). More on Argo CD docs.

### Step 8: Create an Application from a Git Repository

There are two ways to create application. It can be either through CLI or by UI. For the moment we are going to use CLI mode to create our apps. You can choose to use UI mode as well. For the demo purpose, we are going to use a guestbook application available on GitHub repo. To create guestbook application, you need to run below command.

`argocd app create guestbook --repo https://github.com/argoproj/argocd-example-apps.git --path guestbook --dest-server https://kubernetes.default.svc --dest-namespace default`

### Step 9: Sync the Application

Once the application is created you can view the status by using argocd app get guestbook command as shown below.

`argocd app get guestbook`

Initially you will see the status in OutofSync state since the application has yet to be deployed so to sync the application run argocd app sync guestbook command as shown below.

`argocd app sync guestbook`


* Note:

Non-HA:

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.12/manifests/install.yaml
```

HA:

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.12/manifests/ha/install.yaml
```

Password login:

`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo`

Nếu bạn muốn update lại password thì ta làm như sau, trước tiên chạy câu lệnh:

`argocd login <ARGOCD_SERVER>`

`argocd account update-password`

`https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli`












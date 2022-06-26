### Minukube

```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/

minikube start

minikube delete
```

Docker, the software, provides a way of packaging up an application and all of its dependencies, including those at the operating system level, into what is known as a container. Thus, it acts as a lightweight kind of virtual machine.

However, they do not run as a full virtual machine - they run like a normal process. For technically-minded folk, it is like a ‘chroot’ environment. Hence they are much more lightweight, and thus:

They can be started more quickly.
More of them can be run at once on one machine.
Modern containers like Docker-based ones all follow the immutable servers principle, hence a running instance is much easier to replace if it goes bad; this makes fault tolerance easier to implement. See my blog posts on the subject of immutable servers.
For the same reason, as well as the fact that they are quicker to start up, they are much easier to scale.
The ability to package up all of an application's dependencies, including operating system libraries allows:

Testing and debugging with the exact same dependencies as the application uses in production.
Thus there is a much less chance of problems coming up in production that could not be detected during development.
Since the deployment artifact is a container image, it does not matter which language or tools are used to create it or that run inside it. It gets deployed the same way. This means less work to create a platform to run the applications.
However, it is not all plain sailing. The fact that, until recently, there were no easy-to-use platforms that you could use to easily run them meant you had to do all the work to run them yourself - which is an order of magnitude more involved than just running servers. This is what an orchestrator like Kubernetes is used for.  
  
  
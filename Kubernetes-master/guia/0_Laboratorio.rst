

Antes de iniciar debemos tener conocimientos en Docker

Introducción
What is Kubernetes?
Why you need Kubernetes and what it can do?
What Kubernetes is not

Kubernetes Components
	Control Plane ó Master
		kube-apiserver
		kube-scheduler
		kube-controller-manager
		kube-cloud-controller-manager
		etcd
	Node ó Worker
		kubelet
		kube-proxy
		container runtime
Arquitecture Conclusion && Discussion
Ver los Tools
Instalar un Container runtime (Docker)
Instalar kubectl
Instalar minikube


cgomeznt@debian:~$ minikube start
😄  minikube v1.23.2 en Debian 10.10
✨  Controlador docker seleccionado automáticamente
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Descargando Kubernetes v1.22.2 ...
    > gcr.io/k8s-minikube/kicbase: 355.39 MiB / 355.40 MiB  100.00% 99.25 KiB p
    > preloaded-images-k8s-v13-v1...: 511.84 MiB / 511.84 MiB  100.00% 115.18 K
🔥  Creando docker container (CPUs=2, Memory=2200MB) ...
❗  This container is having trouble accessing https://k8s.gcr.io
💡  To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
🐳  Preparando Kubernetes v1.22.2 en Docker 20.10.8...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Complementos habilitados: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
cgomeznt@debian:~$ 




		



Implementar un clúster HA Kubernetes en CentOS7
==================

En este laboratorio vamos a trabajar con tres (3) maquinas:

* un (1) Master

* dos (2) Worker

Nuestro despliegue consiste de los siguientes componentes: 

* Kubernetes

* Etcd

* Docker

* Flannel Network

* Dashboard

* Heapster

Arquitectura
++++++++++++++++

* k8-master01 - 192.168.1.20

* k8-worker01 - 192.168.1.21

* k8-worker02 - 192.168.1.22

Preparando los servidores
+++++++++++++++++++++++++++

Las siguientes tareas se debe realizar en todos los servidores (Master y Worker)


**Deshabilitar Selinux**::

	# setenforce 0
	# sed -i -e 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config

**Deshabilitar swap**::

	# swapoff -a
	# sed -i -e 's/^.*swap/#&/' /etc/fstab

**Deshabilitar firewalld**::

	# systemctl stop firewalld
	# systemctl disable firewalld

**Reiniciar servidores y verificar selinux**::

	# shutdown -r now
	# sestatus

**Editar el archivo /etc/hosts**::

	192.168.1.20	k8-master01
	192.168.1.21	k8-worker02
	192.168.1.22	k8-worker03

**Instalar NTP**::

	# yum install -y ntp
	# systemctl start ntpd
	# systemctl enable ntpd

**Establecer la timezone**::

	# timedatectl set-timezone America/Caracas

**Habilitar módulo br_netfilter**::

	# lsmod | grep br_netfilter
	# modprobe br_netfilter
	# echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
	# lsmod | grep br_netfilter 

**Pre-Requisitos**::

	# yum install -y yum-utils device-mapper-persistent-data lvm2

**Agregar Docker repository**::

	# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

**Instalar Docker CE**::

	# yum update && yum install docker-ce docker-ce-cli

Cambiar el CgroupDrive al docker para que sea Systemd. Crear el directorio de /etc/docker si no existe::

	# mkdir /etc/docker

Configurar el servicio o daemon.::

	# cat > /etc/docker/daemon.json <<EOF
	{
	"exec-opts": ["native.cgroupdriver=systemd"],
	"log-driver": "json-file",
	"log-opts": {
	"max-size": "100m"
	},
	"storage-driver": "overlay2",
	"storage-opts": [
	"overlay2.override_kernel_check=true"
	]
	}
	EOF

	# mkdir -p /etc/systemd/system/docker.service.d

Lo de arriba es para cambiar el cgroupdrive al docker tal como lo recomienda Kubernetes, pero no me funciono.
Ejecute esto y sip me funciono, editar el::

	vi /usr/lib/systemd/system/docker.service
	ExecStart=/usr/bin/dockerd --exec-opt native.cgroupdriver=systemd

	systemctl daemon-reload

	systemctl restart docker

y cuando consulta el systemctl status docker se puede ver::

	21036 /usr/bin/dockerd --exec-opt native.cgroupdriver=systemd

Reiniciar Docker::

	systemctl daemon-reload
	systemctl restart docker
	systemctl enable docker
	systemctl status docker

Consultamos el cgroupfs de docker y vemos que ya esta bien::

	# docker info | grep -i cgroup

Probar Docker::

	# docker run hello-world

**Instalar kubelet, kubeadm, kubectl**::

	# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
	[kubernetes]
	name=Kubernetes
	baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
	enabled=1
	gpgcheck=1
	repo_gpgcheck=1
	gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
		https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
	EOF

Instalación de las herramientas de Kubernete::

	# yum install -y kubelet kubeadm kubectl

Añadimos Kubernetes al cgroup::

	# sed -i 's/cgroup-driver=systemd/cgroup-driver=cgroupfs/g' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

Reiniciamos el Kubernete::

	systemctl daemon-reload
	systemctl enable kubelet
	systemctl start kubelet

Si te fijas el kubelet no inicia, esta con errores, pero luego de inicializar el cluster de kubernete con kubeadm init, se corrige.

No olviden ejecutar, para que puedas ver las consulta del cluster o de los nodos con::

	# kubectl cluster-info
	
	# kubectl get nodes

	export KUBECONFIG=/etc/kubernetes/admin.conf


**Hasta aquí es igual tanto para masters como para workers**




Como dar de baja un Cluster Kubernetes
====================================

Esto se realizo en un kubernetes Version:"v1.19.8", con CentOS 7



Dar de baja los Master
========================

::

	etcdctl del "" --prefix

	kubeadm reset -f

	systemctl stop docker ; systemctl stop kubelet ; systemctl stop etcd

	systemctl start docker ; systemctl start etcd; systemctl start kubelet 

	netstat -natp | grep -i listen

	for i in $(netstat -natp | grep -i listen | grep kube |  awk '{print $7}' | awk -F"/" '{print $1}' ) ; do kill -9 $i ;done

	rm -rf /etc/cni /etc/kubernetes /var/lib/dockershim /var/lib/etcd /var/lib/kubelet /var/run/kubernetes ~/.kube/*

	ifconfig cni0 down ; ifconfig flannel.1 down ; ifconfig docker0 down

	cat /etc/hosts

	systemctl restart ntpd && date

	etcd --version
		etcd Version: 3.3.11
		Git SHA: 2cf9e51
		Go Version: go1.10.3
		Go OS/Arch: linux/amd64

	rpm -qa | grep kube
		kubernetes-cni-0.8.7-0.x86_64
		kubelet-1.19.8-0.x86_64
		kubectl-1.19.8-0.x86_64
		kubeadm-1.19.8-0.x86_64

	yum remove docker-client-1.13.1-103.git7f2769b.el7.centos.x86_64 docker-1.13.1-103.git7f2769b.el7.centos.x86_64 docker-common-1.13.1-103.git7f2769b.el7.centos.x86_64

	yum remove kubeadm-1.19.8-0  kubectl-1.19.8-0  kubelet-1.19.8-0  --disableexcludes=kubernete

	export https_proxy=
	export http_proxy=
	export ftp_proxy=

	docker info | grep HTTP

	vi /etc/systemd/system/docker.service.d/
	[Service]
	Environment="HTTP_PROXY=http://10.132.0.10:8080/"
	Environment="HTTPS_PROXY=http://10.132.0.10:8080/"
	Environment="NO_PROXY=localhost,127.0.0.1,10.134.0.0/16,10.96.0.0/12,10.244.0.0/16,172.17.0.0/16"

https://www.sbarjatiya.com/notes_wiki/index.php/HTTP_proxy_configuration_for_Docker_on_CentOS_7
https://www.codetd.com/en/article/12636173




Dar de baja los Worker
========================

::


	kubeadm reset -f

	systemctl stop docker ; systemctl stop kubelet

	systemctl start docker ; systemctl start etcd; systemctl start kubelet 

	netstat -natp | grep -i listen

	for i in $(netstat -natp | grep -i listen | grep kube |  awk '{print $7}' | awk -F"/" '{print $1}' ) ; do kill -9 $i ;done

	rm -rf /etc/cni /etc/kubernetes /var/lib/dockershim /var/lib/etcd /var/lib/kubelet /var/run/kubernetes ~/.kube/*

	ifconfig cni0 down ; ifconfig flannel.1 down ; ifconfig docker0 down

	cat /etc/hosts

	systemctl restart ntpd && date


	rpm -qa | grep kube
		kubernetes-cni-0.8.7-0.x86_64
		kubelet-1.19.8-0.x86_64
		kubectl-1.19.8-0.x86_64
		kubeadm-1.19.8-0.x86_64

	yum remove docker-client-1.13.1-103.git7f2769b.el7.centos.x86_64 docker-1.13.1-103.git7f2769b.el7.centos.x86_64 docker-common-1.13.1-103.git7f2769b.el7.centos.x86_64

	yum remove kubeadm-1.19.8-0  kubectl-1.19.8-0  kubelet-1.19.8-0  --disableexcludes=kubernete

	export https_proxy=
	export http_proxy=
	export ftp_proxy=

	docker info | grep HTTP

	vi /etc/systemd/system/docker.service.d/
	[Service]
	Environment="HTTP_PROXY=http://10.132.0.10:8080/"
	Environment="HTTPS_PROXY=http://10.132.0.10:8080/"
	Environment="NO_PROXY=localhost,127.0.0.1,10.134.0.0/16,10.96.0.0/12,10.244.0.0/16,172.17.0.0/16"

https://www.sbarjatiya.com/notes_wiki/index.php/HTTP_proxy_configuration_for_Docker_on_CentOS_7
https://www.codetd.com/en/article/12636173



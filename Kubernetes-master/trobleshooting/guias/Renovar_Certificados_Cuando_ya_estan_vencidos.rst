Como renovar los certificados luego de estar vencidos.
=====================

Si nos presenta este error al ejecutar algun comando de kubectl::

	# kubectl get pods
	Unable to connect to the server: x509: certificate has expired or is not yet valid

Debemos irnos a la ruta en donde están los certificados y debemos validar con openssl que realmente sea la falla::

	# openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text |grep ' Not '
				Not Before: Oct 24 03:42:35 2020 GMT
				Not After : Jan 29 00:31:14 2021 GMT

Ejecutamos::

	kubeadm alpha certs renew apiserver
	kubeadm alpha certs renew apiserver-kubelet-client
	kubeadm alpha certs renew front-proxy-client

Generamos el kubeconfig::


	kubeadm alpha kubeconfig user --client-name kubernetes-admin --org system:masters > /etc/kubernetes/admin.conf
	kubeadm alpha kubeconfig user --client-name system:kube-controller-manager > /etc/kubernetes/controller-manager.conf
	# instead of $(hostname) you may need to pass the name of the master node as in "/etc/kubernetes/kubelet.conf" file.
	kubeadm alpha kubeconfig user --client-name system:node:$(hostname) --org system:nodes > /etc/kubernetes/kubelet.conf 
	kubeadm alpha kubeconfig user --client-name system:kube-scheduler > /etc/kubernetes/scheduler.conf


Copiamos el nuevo kubernetes-admin kubeconfig file::

	cp /etc/kubernetes/admin.conf ~/.kube/config

Reiniciamos el servidor o matamos los procesos::

	kill -s SIGHUP $(pidof kube-apiserver)
	kill -s SIGHUP $(pidof kube-controller-manager)
	kill -s SIGHUP $(pidof kube-scheduler)

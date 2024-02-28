Configuración del DNS Credicard en el Clusters de Kubernetes
==========================================

Los clústeres creados por Container Engine para Kubernetes incluyen un servidor DNS como un servicio de Kubernetes integrado que se inicia automáticamente.

Container Engine for Kubernetes crea clústeres con CoreDNS como servidor DNS. CoreDNS es un servidor DNS autorizado de uso general que es modular y conectable.

Consultamos los ConfigMaps para ubicar **coredns** tenemos::

	kubectl get configmaps --namespace=kube-system
	NAME                                 DATA   AGE
	coredns                              2      624d
	extension-apiserver-authentication   6      624d
	kube-flannel-cfg                     2      624d
	kube-proxy                           2      624d
	kubeadm-config                       2      624d
	kubelet-config-1.13                  1      624d
	kubelet-config-1.14                  1      302d
	kubelet-config-1.15                  1      281d
	kubelet-config-1.16                  1      204d
	kubernetes-dashboard-settings        1      617d


Consultamos el ConfigMaps de **coredns** y lo respaldamos::

	sudo kubectl get configmaps --namespace=kube-system coredns -o yaml > coredns.yaml
	
Si consultamos el archivo ** coredns.yaml** veremos algo como esto::

	apiVersion: v1
	data:
	  Corefile: |
		.:53 {
			errors
			health
			ready
			kubernetes cluster.local in-addr.arpa ip6.arpa {
			   pods insecure
			   fallthrough in-addr.arpa ip6.arpa
			   ttl 30
			}
			prometheus :9153
			forward . /etc/resolv.conf
			cache 30
			loop
			reload
			loadbalance
		}
	  Corefile-backup: |
		.:53 {
			errors
			health
			ready
			kubernetes cluster.local in-addr.arpa ip6.arpa {
			   pods insecure
			   fallthrough in-addr.arpa ip6.arpa
			   ttl 30
			}
			prometheus :9153
			forward . /etc/resolv.conf
			cache 30
			loop
			reload
			loadbalance
		}
	kind: ConfigMap
	metadata:
	  creationTimestamp: "2019-12-26T16:03:30Z"
	  name: coredns
	  namespace: kube-system
	  resourceVersion: "69973076"
	  selfLink: /api/v1/namespaces/kube-system/configmaps/coredns
	  uid: 42bfd8dc-27f9-11ea-9335-00505694998a


Crear un archivo del tipo "yaml" para agregar un nuevo ConfigMap que apoye el CoreDNS::

	apiVersion: v1
	kind: ConfigMap
	metadata:
	  name: coredns-custom
	  namespace: kube-system
	data:
	  credicard.server: | # All custom server files must have a .server file extension.
	   credicard.com.ve {
		  forward . 10.132.0.61 10.132.0.3
		}
		
Cargar el ConfigMap de apoyo al CoreDNS::

	kubectl apply -f coredns-custom.yaml

Verificar que se hayan aplicado las personalizaciones ingresando::

	kubectl get configmaps --namespace=kube-system coredns-custom -o yaml

Hacer que CoreDNS  recargar ConfigMap ingresando::

	kubectl delete pod --namespace kube-system -l k8s-app=kube-dns


Consultamos nuevamente los ConfigMaps y debemos ver coredns-custom::

	sudo kubectl get configmaps --namespace=kube-system
	NAME                                 DATA   AGE
	coredns                              2      624d
	coredns-custom                       1      79m
	extension-apiserver-authentication   6      624d
	kube-flannel-cfg                     2      624d
	kube-proxy                           2      624d
	kubeadm-config                       2      624d
	kubelet-config-1.13                  1      624d
	kubelet-config-1.14                  1      302d
	kubelet-config-1.15                  1      281d
	kubelet-config-1.16                  1      204d
	kubernetes-dashboard-settings        1      617d
	
	
Ya con esto garantizamos que todos los PODs puedan tener tambien los DNS de Credicard.


Si por alguna razon quiere editar el ConfigMaps creado pude hacer ::

	kubectl -n kube-system edit configmap coredns-custom

Si quiere eliminar el ConfigMap creado ::

	kubectl delete configmap coredns-custom -n kube-system

**NOTA** https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/
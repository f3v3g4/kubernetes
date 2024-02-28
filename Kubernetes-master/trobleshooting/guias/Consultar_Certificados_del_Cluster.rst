Como consultar los certificados
==================================

Para consultar::

	kubeadm alpha certs check-expiration
	
	CERTIFICATE                EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
	admin.conf                 Jan 29, 2023 00:20 UTC   364d            no
	apiserver                  Jan 29, 2023 00:20 UTC   364d            no
	apiserver-kubelet-client   Jan 28, 2023 23:57 UTC   364d            no
	controller-manager.conf    Jan 29, 2023 00:03 UTC   364d            no
	front-proxy-client         Jan 28, 2023 23:57 UTC   364d            no
	scheduler.conf             Jan 29, 2023 00:04 UTC   364d            no


Si presentaran este error::
	failed to load existing certificate apiserver-etcd-client: open /etc/kubernetes/pki/apiserver-etcd-client.crt: no such file or directory

Pueden ejecutar el mismo comando, pero deben saber dónde está el YAML de configuraciones inicial::

	# kubeadm alpha certs check-expiration --config /etc/kubernetes/configuracion/config.yaml
	
	CERTIFICATE                EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
	admin.conf                 Jan 29, 2023 00:20 UTC   364d            no
	apiserver                  Jan 29, 2023 00:20 UTC   364d            no
	apiserver-kubelet-client   Jan 28, 2023 23:57 UTC   364d            no
	controller-manager.conf    Jan 29, 2023 00:03 UTC   364d            no
	front-proxy-client         Jan 28, 2023 23:57 UTC   364d            no
	scheduler.conf             Jan 29, 2023 00:04 UTC   364d            no

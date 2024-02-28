Como renovar los certificados
=====================


Los consultamos primeramente::

	# kubeadm alpha certs check-expiration
	
	[check-expiration] Reading configuration from the cluster...
	[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'

	CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
	admin.conf                 Dec 30, 2022 19:43 UTC   336d                                    no
	apiserver                  Dec 30, 2022 19:43 UTC   336d            ca                      no
	apiserver-kubelet-client   Dec 30, 2022 19:43 UTC   336d            ca                      no
	controller-manager.conf    Dec 30, 2022 19:43 UTC   336d                                    no
	front-proxy-client         Dec 30, 2022 19:43 UTC   336d            front-proxy-ca          no
	scheduler.conf             Dec 30, 2022 19:43 UTC   336d                                    no

	CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
	ca                      Dec 28, 2031 19:43 UTC   9y              no
	front-proxy-ca          Dec 28, 2031 19:43 UTC   9y              no
	
Renovamos todos los certifcados del cluster::

	# kubeadm alpha certs renew all

	[renew] Reading configuration from the cluster...
	[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'

	certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
	certificate for serving the Kubernetes API renewed
	certificate for the API server to connect to kubelet renewed
	certificate embedded in the kubeconfig file for the controller manager to use renewed
	certificate for the front proxy client renewed
	certificate embedded in the kubeconfig file for the scheduler manager to use renewed

Consultamos nuevamente::

	# kubeadm alpha certs check-expiration
	[check-expiration] Reading configuration from the cluster...
	[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'

	CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
	admin.conf                 Jan 28, 2023 18:05 UTC   364d                                    no
	apiserver                  Jan 28, 2023 18:05 UTC   364d            ca                      no
	apiserver-kubelet-client   Jan 28, 2023 18:05 UTC   364d            ca                      no
	controller-manager.conf    Jan 28, 2023 18:05 UTC   364d                                    no
	front-proxy-client         Jan 28, 2023 18:05 UTC   364d            front-proxy-ca          no
	scheduler.conf             Jan 28, 2023 18:05 UTC   364d                                    no

	CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
	ca                      Dec 28, 2031 19:43 UTC   9y              no
	front-proxy-ca          Dec 28, 2031 19:43 UTC   9y              no


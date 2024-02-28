**Habilitar Dashboard Kubernetes**

Para levantar el dashboard de Kubernetes se debe ingresar al servidor via ssh y realizar el siguiente comando ::

 	kubectl proxy --address=0.0.0.0 --accept-hosts='.*'

Ingresar por la siguientes URL ::

	http://10.134.0.243:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login

**NOTA** Este servicio no debe permanecer arriba debido a que este dashboard no requiere un inicio de sesión con usuario y se pueden realizar modificaciones en los trabajos del cluster 
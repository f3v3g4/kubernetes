Error al ejecutar comando kubectl
======================================

Cuando se presente este error, se debe aplicar el troubleshooting en el perfil del usuario que presenta el inconveniente.

	# kubectl get service
	error: You must be logged in to the server (Unauthorized)

La solución es ir al perfil del usuario y ubicar la carpeta ./kube dentro de ella realizamos un respaldo del archivo config,
luego copiamos de /etc/kubernetes/admin.conf dentro de.kube con el nombre config::

	# cp -dp /etc/kubernetes/admin.conf ./config

Luego si podrá realizar el comando.

	# kubectl get service"
	NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
	ccr-pic-backend-iseries1893   NodePort    10.106.91.91     <none>        443:31302/TCP   209d
	ccr-pic-backend-iseries2309   NodePort    10.101.157.12    <none>        443:31202/TCP   56d
	ccr-pic-backend1886           NodePort    10.105.207.140   <none>        443:31301/TCP   210d
	ccr-pic-backend2414           NodePort    10.96.132.141    <none>        443:31201/TCP   3d1h
	ccr-pic-cron1341              NodePort    10.99.173.45     <none>        80:31203/TCP    330d
	ccr-pic-frontend1909          NodePort    10.99.233.149    <none>        443:31300/TCP   205d
	ccr-pic-frontend2395          NodePort    10.102.45.39     <none>        443:31200/TCP   12d
	kubernetes                    ClusterIP   10.96.0.1        <none>        443/TCP         678d



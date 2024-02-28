**Administracion del cluster Kubernetes**


Kubernetes es un sistema de código libre para la automatización del despliegue, ajuste de escala y manejo de aplicaciones en contenedores 

Para ver uso de CPU y  ::

	kubectl top node

Para ver la cantidad de despliegues ejecutándose ::

	kubectl get deployments

Para ver la cantidad de servicios ejecutándose  (Deployments públicos)::

	Kubectl get services 

Para escalar despliegues :: 

	kubectl scale deployment  <nombre del despliegue> --replicas=<Numero de nodos>

Para ver estado de los nodos ::

	kubectl get nodes

Ver estatus y nombre de los pods ::

	kubectl get pods 

Ver logs  de la aplicación ::

	kubectl logs <nombre del pods>

Crear deployment ::

	kubectl create deployment <nombre del deploy> --image=<imagen Docker>

Crear servicio ::

	kubectl create service nodeport $CI_PROJECT_NAME$CI_PIPELINE_ID --tcp=<Puerto del contenedor>:<Puerto de la aplicacion> --node-port=<puerto por cual se publica la aplicación>

Eliminar despliegues ::

	kubectl delete deploy <nombre del deploy>

Eliminar Servicio :: 

	kubectl delete service <nombre del servicio >

Eliminar servicio y deploy ::

	kubectl delete service/<nombre del servicio> deploy/<nombre del deploy>

Levantar interface gráfica Kubernetes ::

	kubectl proxy --address 0.0.0.0 --accept-hosts '.*'

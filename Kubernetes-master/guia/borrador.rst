

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl delete -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml


kubectl rollout restart -n kube-system deployment/coredns
kubectl rollout status -n kube-system deployment/coredns


kubectl delete pod coredns-84698d49bc-j827n -n kube-system

docker ps
ls /etc/cni/net.d/
ls /etc/kubernetes/
ls /etc/kubernetes/pki
ls /etc/kubernetes/manifests


yum install -y kubeadm-1.16.15-0 kubectl-1.16.15-0 kubelet-1.16.15-0 --disableexcludes=kubernetes



kubectl get configMap kubeadm-config -o yaml --namespace=kube-system

kubeadm config print init-defaults --config=/etc/kubernetes/kubeadm-config.yaml



kubectl get pods -n kube-system
kubectl describe nodes


openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text |grep ' Not '

kubelet services logs

kubeadm alpha certs check-expiration 

kubeadm alpha certs renew all

kubeadm alpha certs renew <nombre del Certificado>

 kubeadm alpha certs renew all --config="/etc/kubernetes/kubeadm-config.yaml"


cat /etc/kubernetes/pki/apiserver.crt | openssl x509 -noout –enddate
cat /etc/kubernetes/pki/apiserver-kubelet-client.crt | openssl x509 -noout –enddate
cat /etc/kubernetes/pki/front-proxy-client.crt | openssl x509 -noout –enddate
 
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
kubectl expose deployment hello-minikube --type=NodePort --port=8080
kubectl delete deploy <deploy>
kubectl delete pods <pod>
 
 
sudo systemctl stop kubelet
sudo rm -rf /var/lib/kubelet/pods/*
sudo systemctl start kubelet


/etc/docker/certs.d/10.134.0.252\:4443


kubectl label --overwrite node lcsqaappworkers03 kubernetes.io/role=worker

Cluster Kubernetes con kubeadm en Rocky Linux 9.4
=========================================

https://www.kubeclusters.com/docs/How-to-Deploy-a-Highly-Available-kubernetes-Cluster-with-Kubeadm-on-CentOS7

https://dockerlabs.collabnix.com/kubernetes/beginners/Install-and-configure-a-multi-master-Kubernetes-cluster-with-kubeadm.html

https://phoenixnap.com/kb/install-kubernetes-on-rocky-linux

https://www.linuxtechi.com/install-kubernetes-on-rockylinux-almalinux/

https://computingforgeeks.com/how-to-install-etcd-on-rhel-centos-rocky-almalinux/

https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/

Kubernetes es un orquestador para contenedores de Docker. En otras palabras, Kubernetes es un software o herramienta de código abierto que se utiliza para organizar y administrar contenedores de Docker en clúster ambiente. Kubernetes también se conoce como k8s y fue desarrollado por Google y donado a "Cloud Native Computing foundation”

En Kubernetes con configuración ETCD, tenemos al menos tres (3) nodos Master y varios nodos Worker. Los nodos de clúster se conocen como nodo Worker o Minion. Desde el nodo Master gestionamos el clúster y su nodos que utilizan el comando "kubeadm" y "kubectl".

El clúster de Kubernetes es altamente configurable. Muchos de sus componentes son opcionales. Nuestro despliegue consiste de los siguientes componentes: 
** Kubernetes, Etcd, Docker, Flannel Network, Dashboard y Heapster.**

En este laboratorio se instala la versión de Kubernetes 1.30.2

  
Como estará configurado el laboratorio
++++++++++++++++++++++++++++++++++++++++


+---------------+-----------------------+-----------------------+
|Server Name	|IP Address		|Role			|
+---------------+-----------------------+-----------------------+
|k8-master01	|192.168.1.20		|Master Node		|
|		|			|			|
|k8-master02	|192.168.1.21		|Master Node		|
|		|			|			|
|k8-master03	|192.168.1.22		|Master Node		|
|		|			|			|
|k8-worker01	|192.168.1.30		|Worker Node		|
|		|			|			|
|k8-registry01	|			|			|
+---------------+-----------------------+-----------------------+


Preparando todos los servidores
===============================

Hay algunas configuraciones previas que se deben realizar en cada uno de los servidores de igual forma.

Deshabilitar Selinux
+++++++++++++++++++++
::

	# setenforce 0
	# sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config

Deshabilitar swap
+++++++++++++++++
::

	# swapoff -a
	# sed -i 's/^.*swap/#&/' /etc/fstab

Deshabilitar firewalld
++++++++++++++++++++++++
::

	# systemctl stop firewalld
	# systemctl disable firewalld

Editar el archivo /etc/hosts
+++++++++++++++++++++++++++++++
::

	# vi  /etc/hosts

	192.168.1.20    k8master01
	192.168.1.21    k8master02
	192.168.1.22    k8master03
	192.168.1.23    k8worker01
	
Cambiar los Hostname de cada uno de los servidores
+++++++++++++++++++++++++++++++++++++++++++
En cada uno de los servidores coloque el mismo nombre como lo coloco en el archivo /etc/hosts::

	# hostnamectl set-hostname k8master01
	
	# hostnamectl set-hostname k8master02
	
	# hostnamectl set-hostname k8master03
	 
	# hostnamectl set-hostname k8worker01

Reiniciar servidores y verificar selinux
++++++++++++++++++++++++++++++++++++++++++++
::

	# shutdown -r now

	# sestatus 
	SELinux status:                 disabled

	# systemctl status firewalld
	● firewalld.service - firewalld - dynamic firewall daemon
	   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
	   Active: inactive (dead)
	     Docs: man:firewalld(1)



Instalar NTP
++++++++++++++++
::

	# dnf install -y ntp
	# systemctl start ntpd
	# systemctl enable ntpd

Establecer la timezone
++++++++++++++++++++++
::

	# timedatectl set-timezone America/Caracas


Instalar Docker
++++++++++++++++

La mejor documentación es la oficial de Docker https://docs.docker.com/engine/install/

Esto es como instalar Docker::

	# dnf install -y yum-utils

	# dnf config-manager --add-repo  https://download.docker.com/linux/centos/docker-ce.repo

	# dnf install docker-ce docker-ce-cli containerd.io


Cambiar el cgroupdrive al docker de init a systemd, tal como lo recomienda Kubernetes::


	# vi /usr/lib/systemd/system/docker.service
	ExecStart=/usr/bin/dockerd --exec-opt native.cgroupdriver=systemd


Recargamos el servicio, lo habilitamos y lo iniciamos::

	# systemctl daemon-reload

	# systemctl enable docker

	# systemctl restart docker

Nos aseguramos que Cgroup Driver sea  systemd::

	# docker info | grep -i cgroup
	 Cgroup Driver: systemd
	 Cgroup Version: 1
	 
Docker para Kubernetes debe tener el Storage Drive de overlay2. Para saber si Docker esta utilizando el Driver de overlay2::

	# docker info | grep Storage
	 Storage Driver: overlay2

Hay un bug con  CentOS 7, XFS y el soporte d_type. El soporte d_type debe estar habilitado en el filesystem de XFS, ver los siguientes link, por ejemplo, en donde tenga instalado Docker, en este ejemplo esta en /var/docker::


	# xfs_info /var/docker/
	meta-data=/dev/sdb               isize=512    agcount=4, agsize=1310720 blks
		 =                       sectsz=512   attr=2, projid32bit=1
		 =                       crc=1        finobt=0 spinodes=0
	data     =                       bsize=4096   blocks=5242880, imaxpct=25
		 =                       sunit=0      swidth=0 blks
	naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
	log      =internal               bsize=4096   blocks=2560, version=2
		 =                       sectsz=512   sunit=0 blks, lazy-count=1
	realtime =none                   extsz=4096   blocks=0, rtextents=0




Para más detalle de XFS y d_type ver estos link::

	https://www.thegeekdiary.com/how-to-create-an-xfs-filesystem/

	https://medium.com/@khushalbisht/docker-on-centos-7-with-xfs-filesystem-can-cause-trouble-when-d-type-is-not-supported-64cee61b39ab


Verificamos el status de Docker ::

	# systemctl status docker
	● docker.service - Docker Application Container Engine
	   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
	   Active: active (running) since mar 2021-12-21 21:29:04 -04; 5min ago
	     Docs: https://docs.docker.com
	 Main PID: 1413 (dockerd)
	    Tasks: 8
	   Memory: 32.5M
	   CGroup: /system.slice/docker.service
		   └─1413 /usr/bin/dockerd --exec-opt native.cgroupdriver=systemd -g /var/docker


Realizamos una prueba de Docker::

	# docker run hello-world
	
	Unable to find image 'hello-world:latest' locally
	latest: Pulling from library/hello-world
	2db29710123e: Pull complete 
	Digest: sha256:2498fce14358aa50ead0cc6c19990fc6ff866ce72aeb5546e1d59caac3d0d60f
	Status: Downloaded newer image for hello-world:latest

	Hello from Docker!
	This message shows that your installation appears to be working correctly.

	To generate this message, Docker took the following steps:
	 1. The Docker client contacted the Docker daemon.
	 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
	    (amd64)
	 3. The Docker daemon created a new container from that image which runs the
	    executable that produces the output you are currently reading.
	 4. The Docker daemon streamed that output to the Docker client, which sent it
	    to your terminal.

	To try something more ambitious, you can run an Ubuntu container with:
	 $ docker run -it ubuntu bash

	Share images, automate workflows, and more with a free Docker ID:
	 https://hub.docker.com/

	For more examples and ideas, visit:
	 https://docs.docker.com/get-started/

Instalar y configurar containerd
+++++++++++++++++++++++++++++++++++++++++

Refrescamos la información locar de los repositorios::

  # dnf makecache

Instalamos el containerd.io::

  #  dnf install -y containerd.io

Hacemos un backup del archivo de configuración de containerd::

  # mv /etc/containerd/config.toml /etc/containerd/config.toml.orig

Creamos un nuevo archivo de configuración con el default template::

  # cd /etc/containerd/

  # containerd config default > config.toml 

 vi config.toml

 disabled_plugins = []
 imports = []
 oom_score = 0
 plugin_dir = ""
 required_plugins = []
 root = "/var/lib/containerd"
 state = "/run/containerd"
 temp = ""
 version = 2

 [cgroup]
  path = ""

 [debug]
  address = ""
  format = ""
  gid = 0
  level = ""
  uid = 0

 [grpc]
  address = "/run/containerd/containerd.sock"
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216
  tcp_address = ""
  tcp_tls_ca = ""
  tcp_tls_cert = ""
  tcp_tls_key = ""
  uid = 0

 [metrics]
  address = ""
  grpc_histogram = false

 [plugins]

  [plugins."io.containerd.gc.v1.scheduler"]
    deletion_threshold = 0
    mutation_threshold = 100
    pause_threshold = 0.02
    schedule_delay = "0s"
    startup_delay = "100ms"

  [plugins."io.containerd.grpc.v1.cri"]
    cdi_spec_dirs = ["/etc/cdi", "/var/run/cdi"]
    device_ownership_from_security_context = false
    disable_apparmor = false
    disable_cgroup = false
    disable_hugetlb_controller = true
    disable_proc_mount = false
    disable_tcp_service = true
    drain_exec_sync_io_timeout = "0s"
    enable_cdi = false
    enable_selinux = false
    enable_tls_streaming = false
    enable_unprivileged_icmp = false
    enable_unprivileged_ports = false
    ignore_deprecation_warnings = []
    ignore_image_defined_volumes = false
    image_pull_progress_timeout = "5m0s"
    image_pull_with_sync_fs = false
    max_concurrent_downloads = 3
    max_container_log_line_size = 16384
    netns_mounts_under_state_dir = false
    restrict_oom_score_adj = false
    sandbox_image = "registry.k8s.io/pause:3.8"
    selinux_category_range = 1024
    stats_collect_period = 10
    stream_idle_timeout = "4h0m0s"
    stream_server_address = "127.0.0.1"
    stream_server_port = "0"
    systemd_cgroup = false
    tolerate_missing_hugetlb_controller = true
    unset_seccomp_profile = ""

    [plugins."io.containerd.grpc.v1.cri".cni]
      bin_dir = "/opt/cni/bin"
      conf_dir = "/etc/cni/net.d"
      conf_template = ""
      ip_pref = ""
      max_conf_num = 1
      setup_serially = false

    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "runc"
      disable_snapshot_annotations = true
      discard_unpacked_layers = false
      ignore_blockio_not_enabled_errors = false
      ignore_rdt_not_enabled_errors = false
      no_pivot = false
      snapshotter = "overlayfs"

      [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        privileged_without_host_devices_all_devices_allowed = false
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""
        sandbox_mode = ""
        snapshotter = ""

        [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime.options]

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]

        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          base_runtime_spec = ""
          cni_conf_dir = ""
          cni_max_conf_num = 0
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          privileged_without_host_devices_all_devices_allowed = false
          runtime_engine = ""
          runtime_path = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"
          sandbox_mode = "podsandbox"
          snapshotter = ""

          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            BinaryName = ""
            CriuImagePath = ""
            CriuPath = ""
            CriuWorkPath = ""
            IoGid = 0
            IoUid = 0
            NoNewKeyring = false
            NoPivotRoot = false
            Root = ""
            ShimCgroup = ""
            SystemdCgroup = true

      [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        privileged_without_host_devices_all_devices_allowed = false
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""
        sandbox_mode = ""
        snapshotter = ""

        [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime.options]

     [plugins."io.containerd.grpc.v1.cri".image_decryption]
      key_model = "node"

     [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = ""

      [plugins."io.containerd.grpc.v1.cri".registry.auths]

      [plugins."io.containerd.grpc.v1.cri".registry.configs."10.134.0.252:4443".tls]
        insecure_skip_verify = true
      [plugins."io.containerd.grpc.v1.cri".registry.headers]

      [plugins."io.containerd.grpc.v1.cri".registry.mirrors."10.134.0.252:4443"]
        endpoint = ["https://10.134.0.252:4443"]
    [plugins."io.containerd.grpc.v1.cri".x509_key_pair_streaming]
      tls_cert_file = ""
      tls_key_file = ""

  [plugins."io.containerd.internal.v1.opt"]
    path = "/opt/containerd"

  [plugins."io.containerd.internal.v1.restart"]
    interval = "10s"

  [plugins."io.containerd.internal.v1.tracing"]

  [plugins."io.containerd.metadata.v1.bolt"]
    content_sharing_policy = "shared"

  [plugins."io.containerd.monitor.v1.cgroups"]
    no_prometheus = false

  [plugins."io.containerd.nri.v1.nri"]
    disable = true
    disable_connections = false
    plugin_config_path = "/etc/nri/conf.d"
    plugin_path = "/opt/nri/plugins"
    plugin_registration_timeout = "5s"
    plugin_request_timeout = "2s"
    socket_path = "/var/run/nri/nri.sock"

  [plugins."io.containerd.runtime.v1.linux"]
    no_shim = false
    runtime = "runc"
    runtime_root = ""
    shim = "containerd-shim"
    shim_debug = false

  [plugins."io.containerd.runtime.v2.task"]
    platforms = ["linux/amd64"]
    sched_core = false

  [plugins."io.containerd.service.v1.diff-service"]
    default = ["walking"]

  [plugins."io.containerd.service.v1.tasks-service"]
    blockio_config_file = ""
    rdt_config_file = ""

  [plugins."io.containerd.snapshotter.v1.aufs"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.blockfile"]
    fs_type = ""
    mount_options = []
    root_path = ""
    scratch_file = ""

  [plugins."io.containerd.snapshotter.v1.devmapper"]
    async_remove = false
    base_image_size = ""
    discard_blocks = false
    fs_options = ""
    fs_type = ""
    pool_name = ""
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.native"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.overlayfs"]
    mount_options = []
    root_path = ""
    sync_remove = false
    upperdir_label = false

  [plugins."io.containerd.snapshotter.v1.zfs"]
    root_path = ""

  [plugins."io.containerd.tracing.processor.v1.otlp"]

  [plugins."io.containerd.transfer.v1.local"]
    config_path = ""
    max_concurrent_downloads = 3
    max_concurrent_uploaded_layers = 3

    [[plugins."io.containerd.transfer.v1.local".unpack_config]]
      differ = ""
      platform = "linux/amd64"
      snapshotter = "overlayfs"

 [proxy_plugins]

 [stream_processors]

  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar"

 [stream_processors."io.containerd.ocicrypt.decoder.v1.tar.gzip"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+gzip+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar+gzip"

 [timeouts]
  "io.containerd.timeout.bolt.open" = "0s"
  "io.containerd.timeout.metrics.shimstats" = "2s"
  "io.containerd.timeout.shim.cleanup" = "5s"
  "io.containerd.timeout.shim.load" = "5s"
  "io.containerd.timeout.shim.shutdown" = "3s"
  "io.containerd.timeout.task.state" = "2s"

 [ttrpc]
  address = ""
  gid = 0
  uid = 0

  
Habilitamos y reiniciamos el servicio::

  # systemctl enable --now containerd.service

Creamos el archivo de configuración para los modulos de Kubernetes y agregamos las dos lineas::

  # vi /etc/modules-load.d/k8s.conf
  overlay
  br_netfilter

Agregamos los modulos al sistema con el comando modprobe::

  # modprobe overlay

  # modprobe br_netfilter

Verificamos que los modulos se cargaran en el sistema::

  # lsmod | grep br_netfilter

  # lsmod | grep overla

Reiniciamos el contenedord::

  # systemctl restart containerd

Crear el archivo /etc/sysctl.d/k8s.conf con el siguiente contenido y luego ejecutar el comando
+++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

	# vi /etc/sysctl.d/k8s.conf
	vm.dirty_expire_centisecs = 500
	vm.swappiness = 10
	net.ipv4.conf.all.forwarding=1
	net.bridge.bridge-nf-call-iptables = 1
	net.bridge.bridge-nf-call-ip6tables = 1
	kernel.pid_max = 4194303

	# sysctl -p /etc/sysctl.d/k8s.conf o sysctl --system

Explícitamente el modulo br_netfilter esta cargado, lo consultamos para estar 100% seguros::

	# lsmod | grep br_netfilter
	br_netfilter           22256  0 
	bridge                151336  1 br_netfilter


Instalar kubelet, kubeadm, kubectl
++++++++++++++++++++++++++++++++
::

  cat <<\EOF > /etc/yum.repos.d/kubernetes.repo
  [kubernetes]
  name=Kubernetes
  baseurl=https://pkgs.k8s.io/core:/stable:/v1.33/rpm/
  enabled=1
  gpgcheck=1
  gpgkey=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/repodata/repomd.xml.key
  exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
  EOF

Listamos todas las versiones de kubeadm, kubelet, kubectl, por si queremos escoger una en particular::

	# dnf list --showduplicates kubeadm --disableexcludes=kubernetes
	
	# dnf list --showduplicates kubelet --disableexcludes=kubernetes
	
	# dnf list --showduplicates kubectl --disableexcludes=kubernetes
	
Instalamos los paquetes kubeadm, kubelet, kubectl y en este ejemplo utilizamos la versión 1.30.2::

	# dnf install -y kubeadm.x86_64  kubectl.x86_64  kubelet.x86_64  --disableexcludes=kubernetes
	

Solo habilitamos el servicio kubelet, NO se debe iniciar porque sino tendrá errores::

	# systemctl daemon-reload
	
	# systemctl enable --now kubelet


Completar comando docker kubeadm kubectl
+++++++++++++++++++++++++++++++++++++++++++

Esto consume muchos recursos en el bash, solo se debería bajo una evaluación::

	# dnf install bash-completion
	source /usr/share/bash-completion/bash_completion
	Desloguea y vuelve a ingresar al perfil. Prueba con:
	type _init_completion
	kubectl completion bash >/etc/bash_completion.d/kubectl
	kubeadm completion bash >/etc/bash_completion.d/kubeadm
	source /usr/share/bash-completion/completions/docker
	

Crear certificados (ejecutar en todos los nodos)
++++++++++++++++++++++++++++++++++++++++++++++++++

Vamos a descargar los PKI and TLS toolkit. Cloud Flare SSL tool genera los diferentes certificados, Kubernetes client, kubectl, para manejar el Kubernetes cluster::

	# curl -s -L -o /usr/local/bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64

	# curl -s -L -o /usr/local/bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64

	# chmod +x /usr/local/bin/cfssl*

	
Verificamos la instalación y que versión es::

	$ cfssl version

**Hasta aquí es igual tanto para Master como para Workers**

Instalar y configurar Etcd (Todos los Master)
+++++++++++++++++++++++++++++++++++++++++++++

etcd es un almacén de valores clave coherente y de alta disponibilidad que se utiliza como almacén de respaldo de Kubernetes para todos los datos del clúster. 

Si su clúster de Kubernetes usa etcd como su almacén de respaldo, asegúrese de tener un plan de respaldo para esos datos. https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster

Puede encontrar información detallada sobre etcd en la documentación oficial.
El numero minimo de nodos es tres (3)

Prepara el area de trabajo de etcd (Todos los Master)
++++++++++++++++++++++++++++++++++++
::

	# mkdir -p /etc/kubernetes/pki/etcd && cd /etc/kubernetes/pki/etcd

Crear la ca y su csr para los certificados (En el Master01)
++++++++++++++++++++++++++++++++++++++++++++

Crear los archivos **ca-config.json** y **ca-csr.json** en /etc/kubernetes/pki/etcd para crear los certificados::

	# vi ca-config.json
	{
	"signing": {
	"default": {
	"expiry": "43800h"
	},
	"profiles": {
	"server": {
	"expiry": "43800h",
	"usages": [
	"signing",
	"key encipherment",
	"server auth",
	"client auth"
	]
	},
	"client": {
	"expiry": "43800h",
	"usages": [
	"signing",
	"key encipherment",
	"client auth"
	]
	},
	"peer": {
	"expiry": "43800h",
	"usages": [
	"signing",
	"key encipherment",
	"server auth",
	"client auth"
	]
	}
	}
	}
	}

::

	# vi ca-csr.json
	{
	"CN": "etcd",
	"key": {
	"algo": "rsa",
	"size": 2048
	}
	}

Generar los certificados (En el Master01)
++++++++++++++++++++++
::

	# cd /etc/kubernetes/pki/etcd
	# /usr/local/bin/cfssl gencert -initca ca-csr.json | /usr/local/bin/cfssljson -bare ca -

Al ejecutar el comando se generan 3 archivos en /etc/kubernetes/pki/etcd::

	ca.pem
	ca-key.pem
	ca.csr


Crear el archivo de configuración para el certificado cliente. (En el Master01)
++++++++++++++++++++++++++++++++++

Para esto crear el archivo /etc/kubernetes/pki/etcd/client.json con el siguiente contenido::

	vi client.json
	{
	"CN": "client",
	"key": {
	"algo": "ecdsa",
	"size": 256
	}
	}

Crear el certificado cliente (En el Master01) 
++++++++++++++++++++++++++++++++++++++++++++++

En /etc/kubernetes/pki/etcd ejecutar::

	# /usr/local/bin/cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client client.json | /usr/local/bin/cfssljson -bare client


Al ejecutar el comando se generan 3 archivos en /etc/kubernetes/pki/etcd::

	client.csr
	client-key.pem
	client.pem

Copiar los certificados (A todos los Master)
+++++++++++++++++++++++++++++++++++++++++++

En el resto de los nodos Master, copiar en la carpeta /etc/kubernetes/pki/etcd los siguientes archivos desde el Master01::

	ca.pem
	ca-key.pem
	client.pem
	client-key.pem
	ca-config.json

Los copiamos así::

	scp ca.pem ca-key.pem client.pem client-key.pem ca-config.json root@192.168.1.21:/etc/kubernetes/pki/etcd/
	scp ca.pem ca-key.pem client.pem client-key.pem ca-config.json root@192.168.1.22:/etc/kubernetes/pki/etcd/ 

Crear el archivo de configuración config.json (En todos los Master)
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

En cada nodo master ejecutar los siguientes comandos::

	# /usr/local/bin/cfssl print-defaults csr > /etc/kubernetes/pki/etcd/config.json

Este comando genera el archivo config.json en /etc/kubernetes/pki/etcd::

	config.json
	{
	"CN": "example.net",
	"hosts": [
	"example.net",
	"www.example.net"
	],"key": {
	"algo": "ecdsa",
	"size": 256
	},
	"names": [
	{
	"C": "US",
	"L": "CA",
	"ST": "San Francisco"
	}
	]
	}

En cada nodo de los Master vamos a ejecutar lo siguiente para que el archivo **config.json** quede con los valores correspondientes::

	# export PRIVATE_IP=$(ip addr show ens192 | grep -Po 'inet \K[\d.]+') && export PEER_NAME=$(hostname)
	# sed -i '0,/CN/{s/example\.net/'"$PEER_NAME"'/}' /etc/kubernetes/pki/etcd/config.json
	# sed -i 's/www\.example\.net/'"$PRIVATE_IP"'/' /etc/kubernetes/pki/etcd/config.json
	# sed -i 's/example\.net/'"$PEER_NAME"'/' /etc/kubernetes/pki/etcd/config.json

El objetivo de los comandos anteriores es configurar el archivo config.json con la ip y nombre del nodo master.

Luego edite manualmente el archivo config.json (C: país, L: estado, ST: ciudad) según su ubicación.::

	# vi config.json
	{
	"CN": "k8master01",
	"hosts": [
	"k8master01",
	"192.168.1.20"
	],
	"key": {
	"algo": "ecdsa",
	"size": 256
	},
	"names": [
	{
	"C": "VE",
	"L": "DC",
	"ST": "CCS"
	}
	]
	}

Crear los certificados de Server y Peer (En todos los Master)
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Esto se debe ejecutar en cada uno de los Master en la siguiente ruta /etc/kubernetes/pki/etcd/::

	/usr/local/bin/cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server config.json | /usr/local/bin/cfssljson -bare server

El comando anterior genera los siguientes archivos en /etc/kubernetes/pki/etcd::

	server.csr
	server-key.pem
	server.pem

Esto se debe ejecutar en cada uno de los Master en la siguiente ruta /etc/kubernetes/pki/etcd/::

	/usr/local/bin/cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=peer config.json | /usr/local/bin/cfssljson -bare peer

El comando anterior genera los siguientes archivos en /etc/kubernetes/pki/etcd::

	peer.csr
	peer-key.pem
	peer.pem

Instalar y configurar ETCD (En todos los Master)
++++++++++++++++++++++++++++++++++++++++++++++++

::

  # ETCD_RELEASE=$(curl -s https://api.github.com/repos/etcd-io/etcd/releases/latest|grep tag_name | cut -d '"' -f 4)
  # echo $ETCD_RELEASE
  # wget https://github.com/etcd-io/etcd/releases/download/${ETCD_RELEASE}/etcd-${ETCD_RELEASE}-linux-amd64.tar.gz

  # tar xvf etcd-${ETCD_RELEASE}-linux-amd64.tar.gz
  # cd etcd-${ETCD_RELEASE}-linux-amd64
  # mv etcd* /usr/local/bin 
  # ls /usr/local/bin 
  etcd  etcdctl  etcdutl

  #  etcd --version
  etcd Version: 3.5.2
  Git SHA: 99018a77b
  Go Version: go1.16.3
  Go OS/Arch: linux/amd64

  # etcdctl version
  etcdctl version: 3.5.2
  API version: 3.5

  # etcdutl version
  etcdutl version: 3.5.2
  API version: 3.5

  Configuramos el ETCD como un servicio::

  # cd
  # mkdir -p /var/lib/etcd/
  # mkdir /etc/etcd

  # groupadd --system etcd
  # useradd -s /sbin/nologin --system -g etcd etcd

  # chown -R etcd:etcd /var/lib/etcd/
  # chmod 0775 /var/lib/etcd/
                     
  # touch /etc/etcd.env
  # export PRIVATE_IP=$(ip addr show ens192 | grep -Po 'inet \K[\d.]+') && export PEER_NAME=$(hostname)
  # echo "PEER_NAME=${PEER_NAME}" >> /etc/etcd.env
  # echo "PRIVATE_IP=${PRIVATE_IP}" >> /etc/etcd.env


                   
El objetivo de los comandos anteriores es instalar etcd y crear el archivo /etc/etcd.env con los
valores PEER_NAME y PRIVATE_IP en los nodos master. Lo consultamos en cada uno de los Master y debe tener los datos de cada uno de ellos::

	cat /etc/etcd.env
	PEER_NAME=k8master01
	PRIVATE_IP=192.168.1.20

Configuración de variables de entorno para la administración básica de ETCD (ETCDCTL) en los tres (3) masters (todos), crear el archivo /etc/profile.d/etcd.sh con el siguiente contenido::

	vi /etc/profile.d/etcd.sh

	export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/client.pem
	export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/client-key.pem
	export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.pem
	export ETCDCTL_ENDPOINTS=https://192.168.1.20:2379,https://192.168.1.21:2379,https://192.168.1.22:2379
	export ETCDCTL_API=3

Iniciamos todas las variables::

# source /etc/profile.d/etcd.sh

En cada nodo master generar el archivo etcd.service en /etc/systemd/system/ con el siguiente contenido:

En el Master01::

	# vi /etc/systemd/system/etcd.service

	[Unit]
	Description=etcd
	Documentation=https://github.com/coreos/etcd
	Conflicts=etcd.service
	Conflicts=etcd2.service
	[Service]
	EnvironmentFile=/etc/etcd.env
	Type=notify
	Restart=always
	RestartSec=5s
	LimitNOFILE=40000
	TimeoutStartSec=0
	ExecStart=/usr/local/bin/etcd \
	--name=k8master01 \
	--data-dir=/var/lib/etcd \
	--listen-client-urls=https://192.168.1.20:2379 \
	--advertise-client-urls=https://192.168.1.20:2379 \
	--listen-peer-urls=https://192.168.1.20:2380 \
	--initial-advertise-peer-urls=https://192.168.1.20:2380 \
	--cert-file=/etc/kubernetes/pki/etcd/server.pem \
	--key-file=/etc/kubernetes/pki/etcd/server-key.pem \
	--client-cert-auth \
	--trusted-ca-file=/etc/kubernetes/pki/etcd/ca.pem \
	--peer-cert-file=/etc/kubernetes/pki/etcd/peer.pem \
	--peer-key-file=/etc/kubernetes/pki/etcd/peer-key.pem \
	--peer-client-cert-auth --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.pem \
	--initial-cluster=k8master01=https://192.168.1.20:2380,k8master02=https://192.168.1.21:2380,k8master03=https://192.168.1.22:2380 \
	--initial-cluster-token my-etcd-token \
	--initial-cluster-state new
	[Install]
	WantedBy=multi-user.target

En el Master02::

	# vi /etc/systemd/system/etcd.service

	[Unit]
	Description=etcd
	Documentation=https://github.com/coreos/etcd
	Conflicts=etcd.service
	Conflicts=etcd2.service
	[Service]
	EnvironmentFile=/etc/etcd.env
	Type=notify
	Restart=always
	RestartSec=5s
	LimitNOFILE=40000
	TimeoutStartSec=0
	ExecStart=/usr/local/bin/etcd \
	--name=k8master02 \
	--data-dir=/var/lib/etcd \
	--listen-client-urls=https://192.168.1.21:2379 \
	--advertise-client-urls=https://192.168.1.21:2379 \
	--listen-peer-urls=https://192.168.1.21:2380 \
	--initial-advertise-peer-urls=https://192.168.1.21:2380 \
	--cert-file=/etc/kubernetes/pki/etcd/server.pem \
	--key-file=/etc/kubernetes/pki/etcd/server-key.pem \
	--client-cert-auth \
	--trusted-ca-file=/etc/kubernetes/pki/etcd/ca.pem \
	--peer-cert-file=/etc/kubernetes/pki/etcd/peer.pem \
	--peer-key-file=/etc/kubernetes/pki/etcd/peer-key.pem \
	--peer-client-cert-auth --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.pem \
	--initial-cluster=k8master01=https://192.168.1.20:2380,k8master02=https://192.168.1.21:2380,k8master03=https://192.168.1.22:2380 \
	--initial-cluster-token my-etcd-token \
	--initial-cluster-state new
	[Install]
	WantedBy=multi-user.target



En el Master03::

	# vi /etc/systemd/system/etcd.service

	[Unit]
	Description=etcd
	Documentation=https://github.com/coreos/etcd
	Conflicts=etcd.service
	Conflicts=etcd2.service
	[Service]
	EnvironmentFile=/etc/etcd.env
	Type=notify
	Restart=always
	RestartSec=5s
	LimitNOFILE=40000
	TimeoutStartSec=0
	ExecStart=/usr/local/bin/etcd \
	--name=k8master03 \
	--data-dir=/var/lib/etcd \
	--listen-client-urls=https://192.168.1.22:2379 \
	--advertise-client-urls=https://192.168.1.22:2379 \
	--listen-peer-urls=https://192.168.1.22:2380 \
	--initial-advertise-peer-urls=https://192.168.1.22:2380 \
	--cert-file=/etc/kubernetes/pki/etcd/server.pem \
	--key-file=/etc/kubernetes/pki/etcd/server-key.pem \
	--client-cert-auth \
	--trusted-ca-file=/etc/kubernetes/pki/etcd/ca.pem \
	--peer-cert-file=/etc/kubernetes/pki/etcd/peer.pem \
	--peer-key-file=/etc/kubernetes/pki/etcd/peer-key.pem \
	--peer-client-cert-auth --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.pem \
	--initial-cluster=k8master01=https://192.168.1.20:2380,k8master02=https://192.168.1.21:2380,k8master03=https://192.168.1.22:2380 \
	--initial-cluster-token my-etcd-token \
	--initial-cluster-state new
	[Install]
	WantedBy=multi-user.target


iniciar el servicio etcd (En todos los Master)
++++++++++++++++++++++++++++++++++++++

Ejecutar los siguientes comandos en cada nodo master, comenzando por el nodo k8master01 para iniciar el servicio etcd::

	# systemctl daemon-reload && systemctl enable etcd
	# systemctl start etcd


Cuando se inicia el servicio con el comando start el master01 no emitirá respuesta hasta que algún otro nodo inicie el servicio etcd con el mismo comando start, es decir, se debe hacer simultáneamente en todos los Master::

	# systemctl status etcd

Verificar la salud del cluster ETCD
++++++++++++++++++++++++++++++++

Con el siguiente comando::

  # etcdctl endpoint health
  https://192.168.1.20:2379 is healthy: successfully committed proposal: took = 5.08282ms
  https://192.168.1.21:2379 is healthy: successfully committed proposal: took = 7.454684ms
  https://192.168.1.212:2379 is healthy: successfully committed proposal: took = 3.454214ms

Verificar los miembros del cluster ETCD
+++++++++++++++++++++++++++++++++++++++++++

Con el siguiente comando::

  # etcdctl member list
  cbac022a76f0f30, started, Lrkdevcrediappma02, https://192.168.1.20:2380, https://192.168.1.20:2379, false
  368ad2b7a4ad8bca, started, Lrkdevcrediappma03, https://192.168.1.21:2380, https://192.168.1.21:2379, false
  9214feb972f3793c, started, Lrkdevcrediappma01, https://192.168.1.22:2380, https://192.168.1.22:2379, false


Configuración de balanceo de ETCD
======================================

Configuracion del Cluster con kubeadm (En todos los Master)
+++++++++++++++++++++++++++++++++++++

Se comienza con el Master01 (k8master01). Crear el directorio /etc/kubernetes/configuration y en el mismo directorio el archivo config.yaml Revisar la versión de kubernetes y colocar la correspondiente en el archivo config.yaml::

  # mkdir /etc/kubernetes/configuration && cd /etc/kubernetes/configuration

  # vi config.yaml 
	apiServer:
  	  certSANs:
          - 10.134.4.172
          extraArgs:
   	    apiserver-count: "3"
            authorization-mode: Node,RBAC
  	 timeoutForControlPlane: 4m0s
       apiVersion: kubeadm.k8s.io/v1beta3
       certificatesDir: /etc/kubernetes/pki
       clusterName: kubernetes
       controlPlaneEndpoint: ""
       controllerManager: {}
       dns:
         imageRepository: k8s.m.daocloud.io/coredns
       etcd:
         external:
           caFile: /etc/kubernetes/pki/etcd/ca.pem
           certFile: /etc/kubernetes/pki/etcd/client.pem
           endpoints:
       - https://10.134.4.172:2379
       - https://10.134.4.173:2379
       - https://10.134.4.174:2379
          keyFile: /etc/kubernetes/pki/etcd/client-key.pem
      imageRepository: k8s.gcr.io
      kind: ClusterConfiguration
      kubernetesVersion: v1.30.2
      networking:
        dnsDomain: cluster.local
        podSubnet: 10.244.0.0/16
        serviceSubnet: 10.96.0.0/12
     scheduler: {}
   ---
   kind: KubeletConfiguration
   apiVersion: kubelet.config.k8s.io/v1beta1
   cgroupDriver: systemd



Este archivo sera el mismo a utilizar en todos los Master, sin realizarle modificaciones.


En el Master01 k8master01 realizar la copia del archivo **config.yaml** al resto de los Master::

	# cd /etc/kubernetes/configuration

	# scp config.yaml root@192.168.1.21:/etc/kubernetes/configuration/

	# scp config.yaml root@192.168.1.22:/etc/kubernetes/configuration/

**NOTA:** Desde la versión 1.16 debe actualizar el archivo config.yaml al formato nuevo. Ejecute el siguiente comando sobre el archivo antes generado.

Ejecutar este comando sobre el **config.yaml** en todos los Master::

	# kubeadm config migrate --old-config config.yaml --new-config config2.yaml

En el Master01, vamos a iniciar la configuración de Kubernetes. Ejecutar el siguiente comando para inicializar el cluster de Kubernetes con la ayuda del archivo config2.yaml::


	# cd /etc/kubernetes/configuration

	# kubeadm init --config=config2.yaml
	

Tomar nota del token del cluster
++++++++++++++++++++++++++++++++++++
::

	kubeadm join 192.168.20.20:6443 --token y7xngl.gw80syc54qulhe93 \
	--discovery-token-ca-cert-hash
	sha256:50204506d189e72ad8391996c739a04c2088f7e9a528f0c5210f26f524d7b2ec

Ejecutar los siguientes comandos en el Master01 (k8master01)::

	# mkdir -p $HOME/.kube && cp -i /etc/kubernetes/admin.conf $HOME/.kube/config && chown $(id -u):$(id -g) $HOME/.kube/config
	# kubectl get pods -n kube-system
	# kubectl get nodes

Copiar en los nodos master 2 y 3 (k8master02, k8master03) en el directorio /etc/kubernetes/pki/ desde el Master01 los siguientes archivos::

	/etc/kubernetes/pki/ca.crt
	/etc/kubernetes/pki/ca.key
	/etc/kubernetes/pki/sa.key
	/etc/kubernetes/pki/sa.pub

Con los siguientes comandos desde el Master01 (k8master01)::

	# cd /etc/kubernetes/pki
		
	# scp ca.crt ca.key sa.key sa.pub root@192.168.1.21:/etc/kubernetes/pki

	# scp ca.crt ca.key sa.key sa.pub root@192.168.1.22:/etc/kubernetes/pki

Ejecutar los siguientes comandos en los nodos master 2 y 3 (k8master02, k8master03) para iniciar el Kubernetes::

	# kubeadm init --config=config2.yaml

	# mkdir -p $HOME/.kube && cp -i /etc/kubernetes/admin.conf $HOME/.kube/config && chown $(id -u):$(id -g) $HOME/.kube/config

Verificar en los tres (3) nodos master los pods de kubernetes ejecutando el siguiente comando::

	# kubectl get pods -n kube-system


Instalar la red de kubernetes “Flannel” (En el Master01)
++++++++++++++++++++++++++++++++++++

En el nodo 1 master (k8master01) ejecutar la instalacion de **calico* ::

	#kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

Ejecutar el siguiente comando para verificar que los pods “coredns” tengan el status “running”::

	# kubectl get pods -n kube-system

Si el comando anterior se ejecuta desde los nodos master 2 y 3 el resultado debe ser el mismo.

Unir los nodos workers al cluster con el comando JOIN
+++++++++++++++++++++++++++++++++++++++++++++++++++++++

Unir los nodos workers al cluster con el comando JOIN NOTA IMPORTANTE : Si el tiempo transcurrido entre la ejecución del comando “ kubeadm
init –config=config.yaml ” el cual generó un token para ser usado con el comando

“kubeadm join ...” es superior a 24 horas se debe generar un nuevo token ya que los tokens expiran a las 24 horas de haber sido generados.

En el Master01::

	# kubeadm token create --print-join-command


Esto se ejecuta en todos los nodes workers::

	# kubeadm join 192.168.1.20:6443 --token wfr0am.wp65pdoqwdul7ige \
	--discovery-token-ca-cert-hash
	sha256:50204506d189e72ad8391996c739a04c2088f7e9a528f0c5210f26f524d7b2ec

Ejecutar el siguiente comando (en el nodo 1 master) para verificar la incorporación de los nodos workers al cluster::

	# kubectl get nodes

Etiquetar el “ROLE” de los workers ya que por defecto la etiqueta “ROLES” en los NO master es “<none>”::

	# kubectl label nodes k8worker01 node-role.kubernetes.io/worker=worker
	# kubectl label nodes k8worker02 node-role.kubernetes.io/worker=worker
	# kubectl label nodes k8worker03 node-role.kubernetes.io/worker=worker

	# kubectl get nodes

Deploy NGINX container en el cluster
++++++++++++++++++++++++++++++++++++++++++++

Ahora hacemos un deploy de NGINX container, desde un nodo master::

	# kubectl create deployment nginx --image=nginx

Luego de crear el deploy ahora se debe crear el servicio para que este disponible por la Red::

	# kubectl create service nodeport nginx --tcp=80:80

::

	# kubectl get deploy

	# kubectl get service

Colocar la IP de cualquiera de los servidores del CLuster y el puerto es el que nos arroje en el "kubectl get service"

	curl Ip:port

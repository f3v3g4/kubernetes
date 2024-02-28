Como instalar nodejs 
========================

Instalación de Nodejs en CentOS 7::

	sudo yum -y update

	curl -sL https://rpm.nodesource.com/setup_12.x | sudo bash -

	sudo yum clean all && sudo yum makecache fast
	sudo yum install -y gcc-c++ make
	sudo yum install -y nodejs

	Verificamos la versión::
	
	$ node -v

Como instalar Axios
========================
Instalamos axios Libreria utilizada como base HTTP y como cliente tambien.

	$ sudo npm install axios
	$ sudo npm install -g axios












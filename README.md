# 3tier_todo_app

launch Ubuntu instance -t3-medium- 30gb of SSD both master and worker node.Run the script 

-->kubeadm.sh 


-----Kubeadm-------------------Install this script on both Master & worker Node -----------Shell-Script------------------

echo "Step 1: Login with root user and Install Docker ( in Master & Worker Node Both)"
sudo apt-get update -y
sudo apt-get install \
ca-certificates \
curl \
gnupg -y

echo "Add Docker’s official GPG key:"
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg -y

echo "Use the following command to set up the repository:"

echo \
"deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
"$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update -y
echo "To install the latest version, run:"
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

echo "Step 2: Create a file with the name containerd.conf using the command:"
# create the file with root privileges using the vim editor
sudo vim /etc/modules-load.d/containerd.conf <<EOF
i
overlay
br_netfilter
Esc
:wq
EOF

echo "Step 3: Save the file and run the following commands:"
modprobe overlay
modprobe br_netfilter

echo "Step 4: Create a file with the name kubernetes.conf in /etc/sysctl.d folder:"
# create the file with root privileges using the vim editor
sudo vim /etc/sysctl.d/kubernetes.conf <<EOF
i
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
Esc
:wq
EOF

echo "Step 5: Run the commands to verify the changes:"
sudo sysctl --system
sudo sysctl -p

echo "Step 6: Remove the config.toml file from /etc/containerd/ Folder and run reload your system daemon:"
rm -f /etc/containerd/config.toml
systemctl daemon-reload

echo "Step 7: Add Kubernetes Repository:"
apt-get update && apt-get install -y apt-transport-https ca-certificates curl -y
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

echo "Step 8: Disable Swap"
swapoff -a

Step 9: Export the environment variable:
export KUBE_VERSION=1.23.0

echo "Step 10: Install Kubernetes:"
apt-get update -y
apt-get install -y kubelet=${KUBE_VERSION}-00 kubeadm=${KUBE_VERSION}-00 kubectl=${KUBE_VERSION}-00 kubernetes-cni=0.8.7-00
apt-mark hold kubelet kubeadm kubectl
systemctl enable kubelet
systemctl start kubelet

-----Kubeadm-------------------Install this script on both Master & worker Node -----------Shell-Script------------------

master node
-->kubeadm init --kubernetes-version=${KUBE_VERSION}

worker node
--> check the worker node master node is effected or not

--> kubectl get nodes

--> go to docker hub and search for  (mysql)

--> kubectl create cm db-config --from-literal=MYSQL_DATABASE=sqldb

--> kubectl create secret generic db-secret --from-literal=MYSQL_ROOT_PASSWORD=rootpassword

--> kubectl run mysql-pod --image=mysql --dry-run=client -o yaml > mysql-pod.yaml

--> cat mysql-pod.yaml

--> vi mysql-pod.yaml
   
-->(under containers, -images, name write below)
		-> envFrom:
		   - configMapRef:
			name: db-config
	         - secretRef:
			name: db-secret

		:wq -->save it

--> kubectl apply -f mysql-pod.yaml

--> kubectl get all


(Expose the pod)
--> kubectl expose pod mysql-pod --port=3306 --target-port=3306 --name=mysql-svc

--> kubectl get all

------configmap for myphp-admin-----------

--> go to docker hub and search for  (Phpmyadmin)

--> kubectl create cm phpadmin-config --from-literal=PMA_HOST=(copy mysql-svc-cluster-IP) --from-literal=PMA_PORT=3306

--> kubectl create secret generic phpadmin-secret --from-literal=PMA_USER=root ----from-literal=PMA_PASSWORD=rootpassword

--> kubectl run phpadmin-pod --image=phpmyadmin --dry-run=client -o yaml > phpadmin-pod.yaml

--> vi phpadmin-pod.yaml
   
-->(under containers, -images, name write below)
		-> envFrom:
		   - configMapRef:
			name: phpadmin-config
	         - secretRef:
			name: phpadmin-secret

		:wq -->save it

--> kubectl apply -f phpadmin-pod.yaml

--> kubectl get all

(Expose the pod)
--> kubectl expose pod phpadmin-pod --type=NodePort ---port=8099 --target-port=80 --name=phpadmin-svc

--> kubectl get all

--> (copy IPV4:(Nodeport)) --> open in a browser

--> (open phpmyadmin-dashboard page.)

-----------------------------------------------


-->go to Sqldb	
	-> EXPORT the DATA INTO DATABASE

	-> import ->select
	-> choose file 
		--> simple_todo.sql -->select..
	-> Import
	--> click on todo and see the demo data.


-----PHP TO-DO_APP--POD-----------------

--->open ubntu vm and home Dir

--> git clone https://github.com/cloudtarun97/3tier-to-do-app.git

--> cd 3tier-to-do-app

-->ls

--> open index.php (edit the host name and cluster ip address)
	-->vi index.php

-> $mysql_connect ('(mysql-svc-ip)','root','rootpassword','sqldb')
	->save it

-->docker build -t "dockerusrname"/phpwebapp:latest

--> docker images

--> docker login

--> docker images push "dockerusrname"/phpwebapp:latest"


--> kubectl run php-app --image="dockerusrname"/phpwebapp

-->kubectl get all

--> kubectl expose pod php-app --type=NodePort --port=8080 --target-port=80 --name=phpapp-svc

(copy IPV4:nodeport of phpapp run on the browser.)

	

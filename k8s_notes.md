## Kubernetes (K8s)
* The shortform of k8s is came from the charachters between the kubernetes of (K-s)
## Definition of kubernetes
* Kubernetes is a portable, extensible, open source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. It has a large, rapidly growing ecosystem. Kubernetes services, support, and tools are widely available.
## Need for K8s
  ## High Availability (HA):
* When we run our applications in docker container and if the container fails, we need to manually start the container.
* If the node i.e. the machine fails all the containers running on the machine should be re-created on other machine.
* K8s can do both of the above.
## Autoscaling
* Containers don’t scale on their own.
* Scaling is of two types
   * Vertical Scaling
   * Horizontal Scaling
* K8s can do both horizontal and vertical scaling of containers
Zero-Down time Deployments
* K8s can handle deployments with near zero-down time deployments
* K8s can handle rollout (new version) and roll back (undo new version => previous version)
* K8s is described as `"Production grade Container management"`.
## Installation of Kubernetes(k8s)
*  Single Node Installations
    * minikube
    * kind
* On-prem installations
    * kube-admin
* k8s as a Service
    * AKS
    * EKS
    * GKE
## Kube-admin Installation
* Initially k8s used docker as a main container platform and docker used to get special treatment, from k8s 1.24 special treatment is stopped.
* k8s is designed to run any container technology, for this k8s expects container technology to follow k8s interfaces.
* Before install kubernetes we need to install CRI-Dockerd refer here url@https://github.com/Mirantis/cri-dockerd it is only for k8s after 1.24 verions.
## Prerequisets:
* Create 2 or 3 virtual machines with minimum of 2vCPUS and 4GB RAM
* ![preview](images/k8s2.png)
* Installation method (kubeadm) which is something we will be using in on-premises k8s.
* Install docker in all the nodes.
* ![preview](images/k8s3.png)
* ![preview](images/k8s4.png)
* Install CRI-DOCKER in all the nodes, run the commands as root user.
```bash
# Run these commands as root
###Install GO###
wget https://storage.googleapis.com/golang/getgo/installer_linux
chmod +x ./installer_linux
./installer_linux
source ~/.bash_profile

git clone https://github.com/Mirantis/cri-dockerd.git
cd cri-dockerd
mkdir bin
go build -o bin/cri-dockerd
mkdir -p /usr/local/bin
install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
cp -a packaging/systemd/* /etc/systemd/system
sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
systemctl daemon-reload
systemctl enable cri-docker.service
systemctl enable --now cri-docker.socket
```
![preview](images/k8s5.png)
![preview](images/k8s5.1.png)
* Refer here for commands to install kubeadm,kubctl,kubelet url@https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
* Run these commands as root user `sudo apt-get update
* sudo apt-get install -y apt-transport-https ca-certificates curl`
* `sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg`
* `echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list`
* `sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl`
![preview](images/k8s6.1.png)
* To configure the master node we have to follow below commands and wherever we apply these commands that is considered to be as a master node of kubeadm `kubeadm --help`,`kubeadm init --help`
* To configure kubeadm we have to take network policy as flannel that is "10.244.0.0/16" refer here url@https://github.com/flannel-io/flannel"
* ` kubeadm init --pod-network-cidr "10.244.0.0/16" --cri-socket "unix:///var/run/cri-dockerd.sock"` output of after applying the command
* ![preview](images/k8s6.png)
* ![preview](images/k8s7.png)
* To configure we have to follow above commands as regular user so you can exit as root user.
* To setup configure run this command to install flannel `kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml`
* ![preview](images/k8s8.png)
* To attache the nodes to the master run this command by providing CRI-DOCKER in node as a root user `kubeadm join 172.31.85.229:6443 --token bq3k8r.gq6fdcgdaa3oqelt \
        --cri-socket "unix:///var/run/cri-dockerd.sock" \
        --discovery-token-ca-cert-hash sha256:659c7d1ad0262f54bcaa2be1610b251cc064345236d8342a39c3eae0e2ef1617`
![preview](images/k8s9.png)
* In master node run `kubectl get nodes -w`
* ![preview](images/k8s10.png)
# -*- mode: ruby -*-
# vi: set ft=ruby :

# This script to install Kubernetes will get executed after we have provisioned the box 
$script = <<-SCRIPT
# Install docker
echo Installing docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt-get update
apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu
# Install kubernetes
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
# kubelet requires swap off
swapoff -a
# keep swap off after reboot
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo cp /vagrant/swapoff.service /lib/systemd/system/swapoff.service
sudo systemctl enable swapoff.service
# sed -i '/ExecStart=/a Environment="KUBELET_EXTRA_ARGS=--cgroup-driver=cgroupfs"' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
sed -i '0,/ExecStart=/s//Environment="KUBELET_EXTRA_ARGS=--cgroup-driver=cgroupfs"\n&/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
# Get the IP address that VirtualBox has given this VM
IPADDR=`ifconfig eth1 | grep Mask | awk '{print $2}'| cut -f2 -d:`
echo This VM has IP address $IPADDR
# Set up Kubernetes
NODENAME=$(hostname -s)
kubeadm init --apiserver-cert-extra-sans=$IPADDR  --node-name $NODENAME
# Set up admin creds for the vagrant user
echo Copying credentials to /home/vagrant...
sudo --user=vagrant mkdir -p /home/vagrant/.kube
cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config
# Send credentials to external access
echo Sending credentials to external access
cp /home/vagrant/.kube/config /vagrant/credentials.conf
sed -i -e 's~server: https://.*:6443~server: https://'$IPADDR':6443~g' /vagrant/credentials.conf
# Set up Permissive RBAC Permissions
echo creating permissive RBAC permissions to
echo USER: admin
echo PASSWORD: kubelet
sudo kubectl --kubeconfig /etc/kubernetes/admin.conf create clusterrolebinding permissive-binding --clusterrole=cluster-admin --user=admin --user=kubelet --group=system:serviceaccounts
# Start kubectl proxy
echo Starting kubectl proxy
sudo cp /vagrant/kubectlproxy.service /lib/systemd/system/kubectlproxy.service
sudo systemctl enable kubectlproxy.service
# Install a pod network
echo Install a pod network
sudo kubectl --kubeconfig /etc/kubernetes/admin.conf apply -f https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')
# Remove taint to block schedule on master
echo Removing taints
kubectl --kubeconfig /etc/kubernetes/admin.conf taint nodes --all node-role.kubernetes.io/master-
# Create local registry
echo Creating local registry
docker run -d -p 5000:5000 --restart=always --name registry registry:2
# Build Jenkins image
echo Building Jenkins image
docker build -t vagrant:5000/kubernetes-jenkins /vagrant/jenkins
echo Pushing jenkins image to local registry
docker push vagrant:5000/kubernetes-jenkins
# Create Jenkins pod
echo Creating jenkins pod
sudo kubectl --kubeconfig /etc/kubernetes/admin.conf apply -f /vagrant/jenkins/jenkins-deployment.yaml
sudo kubectl --kubeconfig /etc/kubernetes/admin.conf create -f /vagrant/jenkins/jenkins-service.yaml
SCRIPT

Vagrant.configure("2") do |config|
  # Specify your hostname if you like
  # config.vm.hostname = "name"
  config.vm.box = "bento/ubuntu-16.04"
  config.vm.network "private_network", type: "dhcp"
#  config.vm.provision "docker"
  config.vm.provision "shell", inline: $script
  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 2
  end
end

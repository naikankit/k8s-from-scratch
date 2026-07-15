Kubernetes Installation Guide (Containerd Runtime)
Prerequisites
	• Linux (Ubuntu)
	• Root or sudo access
	• Internet connectivity
	• Swap disabled
	• Hostname and networking configured
Step 1: Configure Kernel Parameters
(Optional) Disable IPv6
Create a configuration file:
cat <<EOF | sudo tee /etc/sysctl.d/99-disable-ipv6.conf
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
EOF

 sysctl -p /etc/sysctl.d/99-disable-ipv6.conf
Enable IPv4 Forwarding
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

Load br_netfilter
Load immediately and Load automatically after reboot:
sudo modprobe br_netfilter
echo "br_netfilter" | sudo tee /etc/modules-load.d/k8s.conf

Enable Bridge Networking
Append to the same file:
cat <<EOF | sudo tee -a /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
Apply all changes:
sudo sysctl --system
Reference:
https://kubernetes.io/docs/setup/production-environment/container-runtimes/


Step 2: Install Containerd
Download the latest release:
https://github.com/containerd/containerd/releases
Example:
wget https://github.com/containerd/containerd/releases/download/v2.3.3/containerd-2.3.3-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local containerd-2.3.3-linux-amd64.tar.gz
Reference:
https://containerd.io/docs/2.3/getting-started/#option-2-from-apt-get-or-dnf

Step 3: Install the systemd Service
Check your init system:
ps -p 1
If the output is systemd, install the service file.
Download:
https://github.com/containerd/containerd/releases
sudo mkdir -p /usr/local/lib/systemd/system
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mv containerd.service /usr/local/lib/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
Verify:
systemctl status containerd


Step 4: Install runc
Download:
https://github.com/opencontainers/runc/releases
Example:
wget https://github.com/opencontainers/runc/releases/download/v1.3.1/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc

Step 5: Install CNI Plugins
Create directory:
sudo mkdir -p /opt/cni/bin
Download:
https://github.com/containernetworking/plugins/releases
Example:
wget https://github.com/containernetworking/plugins/releases/download/v1.9.1/cni-plugins-linux-amd64-v1.9.1.tgz
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.9.1.tgz

Step 6: Install crictl
Download:
https://github.com/kubernetes-sigs/cri-tools/releases
Example:
VERSION="v1.36.0"
curl -LO https://github.com/kubernetes-sigs/cri-tools/releases/download/$CRIVERSION/crictl-$VERSION-linux-amd64.tar.gz
sudo tar -C /usr/local/bin -xzf crictl-$VERSION-linux-amd64.tar.gz
rm -f crictl-$VERSION-linux-amd64.tar.gz
Create the configuration:
sudo nano /etc/crictl.yaml
Contents:
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock

Step 7: Configure Containerd
Create the directory:
sudo mkdir -p /etc/containerd
Generate the default configuration:
containerd config default | sudo tee /etc/containerd/config.toml
Enable the systemd cgroup driver:
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
Restart containerd:
sudo systemctl restart containerd


sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.35/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet=1.35.0-1.1 kubeadm=1.35.0-1.1  kubectl=1.35.0-1.1 
apt-cache madison kubeadm

sudo apt-mark hold kubelet kubeadm kubectl
 sudo systemctl enable --now kubelet


Step 8: Initialize the Kubernetes Control Plane
Example:
sudo kubeadm init \
--apiserver-cert-extra-sans=controlplane \
--apiserver-advertise-address=192.168.1.102 \
--pod-network-cidr=10.244.0.0/16 \
--service-cidr=10.128.0.0/16

Step 9: Configure kubectl
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
Verify:
kubectl get nodes

Step 10: Install Flannel CNI only on master
Download:
https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
Apply:
wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
kubectl apply -f kube-flannel.yml
Verify:
kubectl get pods -n kube-flannel

Step 11: Join Worker Nodes
After kubeadm init, copy the generated command.
Example:
kubeadm join <CONTROL_PLANE_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>

Useful Verification Commands
containerd --version
runc --version
crictl version
systemctl status containerd
kubectl get nodes
kubectl get pods -A

Official References
	• Containerd: https://containerd.io/docs/2.3/getting-started/
	• Containerd Releases: https://github.com/containerd/containerd/releases
	• Kubernetes Runtime Setup: https://kubernetes.io/docs/setup/production-environment/container-runtimes/
	• runc Releases: https://github.com/opencontainers/runc/releases
	• CNI Plugins: https://github.com/containernetworking/plugins/releases
	• CRI Tools: https://github.com/kubernetes-sigs/cri-tools/releases
	• Flannel: https://github.com/flannel-io/flannel

From <https://chatgpt.com/c/6a54d769-344c-83e8-9a7c-3109fb1b07ef> 


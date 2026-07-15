Here’s your Kubernetes Installation Guide (Containerd Runtime) formatted neatly in Markdown for a GitHub repo:

# Kubernetes Installation Guide (Containerd Runtime)

## Prerequisites
- Linux (Ubuntu)
- Root or sudo access
- Internet connectivity
- Swap disabled
- Hostname and networking configured

---

## Step 1: Configure Kernel Parameters

### (Optional) Disable IPv6
```bash
cat <<EOF | sudo tee /etc/sysctl.d/99-disable-ipv6.conf
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
EOF

sudo sysctl -p /etc/sysctl.d/99-disable-ipv6.conf

Enable IPv4 Forwarding

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

Load br_netfilter

sudo modprobe br_netfilter
echo "br_netfilter" | sudo tee /etc/modules-load.d/k8s.conf

Enable Bridge Networking

cat <<EOF | sudo tee -a /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system

Reference: Kubernetes Runtime Setup

Step 2: Install Containerd

wget https://github.com/containerd/containerd/releases/download/v2.3.3/containerd-2.3.3-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local containerd-2.3.3-linux-amd64.tar.gz

Reference: Containerd Docs

Step 3: Install the systemd Service

ps -p 1   # Check init system

sudo mkdir -p /usr/local/lib/systemd/system
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mv containerd.service /usr/local/lib/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
systemctl status containerd

Step 4: Install runc

wget https://github.com/opencontainers/runc/releases/download/v1.3.1/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc

Step 5: Install CNI Plugins

sudo mkdir -p /opt/cni/bin
wget https://github.com/containernetworking/plugins/releases/download/v1.9.1/cni-plugins-linux-amd64-v1.9.1.tgz
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.9.1.tgz

Step 6: Install crictl

VERSION="v1.36.0"
curl -LO https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
sudo tar -C /usr/local/bin -xzf crictl-$VERSION-linux-amd64.tar.gz
rm -f crictl-$VERSION-linux-amd64.tar.gz

Create config:

sudo nano /etc/crictl.yaml

Contents:

runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock

Step 7: Configure Containerd

sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restart containerd

Step 8: Install Kubernetes Components

sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.35/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet=1.35.0-1.1 kubeadm=1.35.0-1.1 kubectl=1.35.0-1.1
apt-cache madison kubeadm
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet

Step 9: Initialize the Kubernetes Control Plane

sudo kubeadm init \
--apiserver-cert-extra-sans=controlplane \
--apiserver-advertise-address=192.168.1.102 \
--pod-network-cidr=10.244.0.0/16 \
--service-cidr=10.128.0.0/16

Step 10: Configure kubectl

mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get nodes

Step 11: Install Flannel CNI (Master Only)

wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
kubectl apply -f kube-flannel.yml
kubectl get pods -n kube-flannel

Step 12: Join Worker Nodes

kubeadm join <CONTROL_PLANE_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>

Useful Verification Commands

containerd --version
runc --version
crictl version
systemctl status containerd
kubectl get nodes
kubectl get pods -A

Official References

Containerd Docs

Containerd Releases

Kubernetes Runtime Setup

runc Releases

CNI Plugins

CRI Tools

Flannel


This is now **GitHub-ready** as a `.md` file. You can drop it straight into your repo under `docs/` or root.  

Would you like me to also add a **table of contents** at the top so navigation is easier in GitHub?

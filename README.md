# 🧱 REQUISITOS PREVIOS

- Debian 12 instalado  
- Usuario con permisos `sudo`  
- IP fija: `10.0.0.75`  
- Firewall desactivado o configurado  
- Nombre del host configurado (ej: `hostnamectl set-hostname master-node`)

---

## 1. Actualizar el sistema e instalar dependencias básicas

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release software-properties-common
```

## 2. Desactivar swap (obligatorio para kubelet)
```bash
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab
```

## 3. Cargar módulos del kernel necesarios
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

## 4. Instalar containerd
```bash
  sudo apt install -y containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml >/dev/null
sudo systemctl restart containerd
sudo systemctl enable containerd
```

## 5. Instalar Kubernetes (kubeadm, kubelet, kubectl)
```bash
sudo curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## 6. Inicializar el cluster con kubeadm
```bash
sudo kubeadm init --apiserver-advertise-address=10.0.0.75 --pod-network-cidr=192.168.0.0/16
```

## 7. Configurar kubectl para el usuario actual
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 8. Instalar Calico como CNI
```bash
curl https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```

## 9. Verificar estado del cluster
```bash
kubectl get nodes
kubectl get pods -A
```

## 🔒 (Opcional) Desactivar firewall para evitar dramas
```bash
sudo systemctl stop ufw
sudo systemctl disable ufw
```

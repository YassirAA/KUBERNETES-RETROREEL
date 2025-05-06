
# 🚀 Kubernetes Multi-Node Cluster en Ubuntu 24.04 LTS usando Kubeadm + Calico

Este tutorial describe paso a paso cómo desplegar un clúster multi-nodo de Kubernetes usando `kubeadm`, `containerd` y **Calico como red CNI**, en servidores Ubuntu 24.04 LTS actualizados.

---

## 🔧 Requisitos

- 2 o más máquinas con Ubuntu 24.04 LTS
- Mínimo 2 vCPU y 2 GB de RAM por máquina
- Conectividad entre los nodos
- Acceso root o privilegios `sudo`
- Swap desactivado
- Muy importante tener dos ip's configuradas en nuestra maquina para que funcione correctamente el cluster en mi caso tengo una publica 192.168.0.X y la privada entre solo los nodes 192.168.10.X

---

## 1️⃣ Actualizar sistema e instalar dependencias

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install apt-transport-https curl -y
```
## Instalar y configurar containerd
```bash
sudo apt install containerd -y

sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null

# Cambiar el valor de cgroups a systemd
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

sudo systemctl restart containerd

```
## Instalar kubelet, kubeadm y kubectl
```bash
sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

```
## Desactivar swap
```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

```
## Cargar módulos del kernel necesarios
```bash
sudo modprobe overlay
sudo modprobe br_netfilter

```
## Aplicar parámetros sysctl requeridos por Kubernetes
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system

```
## Inicializar el clúster (solo en el nodo master)
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16

```
## Configurar kubectl (solo en el nodo master)
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```
## Instalar Calico como red CNI (solo en el nodo master)
```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml

```
## Verificar el estado del clúster
```bash
kubectl get nodes
kubectl get pods --all-namespaces

```
## Añadir nodos workers al clúster
```bash
sudo kubeadm join <MASTER_IP>:6443 --token <TOKEN> \
    --discovery-token-ca-cert-hash sha256:<HASH>

```

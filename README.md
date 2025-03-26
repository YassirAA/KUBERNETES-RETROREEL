# KUBERNETES-RETROREEL

## PREPARAR MASTER i WORKER
### Deshabilitar swap
```bash
sudo swapoff -a
sed -i '/swap/d' /etc/fstab
```

### Configurar Kernel
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
sudo modprobe br_netfilter
```

### Configurar el sistema
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```

### Instalar containerd
```bash
sudo yum install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

### Instalar kube ....
```bash
sudo yum install -y kubeadm kubelet kubectl
sudo systemctl enable --now kubelet
```

### Preparar Calico
```bash
(Al master)
# En el master:
kubectl get pods -n kube-system | grep calico
```
```bash
(Al worker)
scp root@172.24.59.241:/etc/kubernetes/admin.conf ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

### Agafar comanda per unir el worker
```bash
(Al master)
kubeadm token create --print-join-command
```


## Verificacio
```bash
kubectl get nodes
kubectl get pods -n kube-system
```

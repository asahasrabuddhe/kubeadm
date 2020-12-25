# KubeADM

Create a network
```bash
gcloud compute networks create kubeadm --subnet-mode custom
```

Create a subnet
```bash
gcloud compute networks subnets create kubernetes \
    --network kubeadm \
    --range 10.240.0.0/24
```

Create Firewall Rules

1. Internal Access
    6443 - API Server
    2379-2380 - etcd server client API
    10250 - Kubelet API
    10251 - Kube Scheduler
    10252 - Kube Controller Manager
    30000-32767 - NodePort Services
    ```bash
    gcloud compute firewall-rules create kubeadm-internal-access \
        --allow tcp:6443,tcp:2379-2380,tcp:10250-10252,tcp:30000-32767 \
        --network kubeadm \
        --source-ranges 10.240.0.0/24,10.200.0.0/16
    ```
2. External Access
    22 - SSH
    80 - HTTP
    443 - HTTPS
    ```bash
    gcloud compute firewall-rules create kubeadm-external-access \
        --allow tcp:22,tcp:80,tcp:443 \
        --network kubeadm \
        --source-ranges 0.0.0.0/0
    ```
    
```bash
for i in 0 1 2; do
  gcloud compute instances create controller-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-2004-lts \
    --image-project ubuntu-os-cloud \
    --machine-type e2-standard-2 \
    --private-network-ip 10.240.0.1${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubeadm,controller
done
```

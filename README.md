## HPC_on_Proxmox
Deploy Debian 12 LXC containers to emulate HPC

Provision on Proxmox

### 1. Create slurm-head (Controller / NFS Master)
```
pct create 130 local:vztmpl/debian-12-standard_12.12-1_amd64.tar.zst \
  --hostname slurm-head \
  --memory 2048 --swap 1024 --cores 2 \
  --ostype debian --unprivileged 0 \
  --features nesting=1,nfs=1 \
  --onboot 0 \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.1.130/24,gw=192.168.1.1 \
  --rootfs local-lvm:8
```
### 2. Create slurm-node1 (Worker 1)
```
pct create 131 local:vztmpl/debian-12-standard_12.12-1_amd64.tar.zst \
  --hostname slurm-node1 \
  --memory 2048 --swap 1024 --cores 2 \
  --ostype debian --unprivileged 0 \
  --features nesting=1,nfs=1 \
  --onboot 0 \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.1.131/24,gw=192.168.1.1 \
  --rootfs local-lvm:8
```
### 3. Create slurm-node2 (Worker 2)
```
pct create 132 local:vztmpl/debian-12-standard_12.12-1_amd64.tar.zst \
  --hostname slurm-node2 \
  --memory 2048 --swap 1024 --cores 2 \
  --ostype debian --unprivileged 0 \
  --features nesting=1,nfs=1 \
  --onboot 0 \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.1.132/24,gw=192.168.1.1 \
  --rootfs local-lvm:8
```
# Start all three containers
```
pct start 130 && pct start 131 && pct start 132
```

## HPC_on_Proxmox
Deploy Debian 12 LXC containers to emulate HPC

Provision on Proxmox

1. Create slurm-head (Controller / NFS Master)
```
pct create 130 local:vztmpl/debian-12-standard_12.7-1_amd64.tar.zst \
  --hostname slurm-head \
  --memory 2048 --swap 1024 --cores 2 \
  --ostype debian --unprivileged 0 \
  --features nesting=1,mount=nfs \
  --onboot 0 \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.1.130/24,gw=192.168.1.1 \
  --rootfs local-lvm:8
```
2. Create slurm-node1 (Worker 1)
```
pct create 131 local:vztmpl/debian-12-standard_12.7-1_amd64.tar.zst \
  --hostname slurm-node1 \
  --memory 2048 --swap 1024 --cores 2 \
  --ostype debian --unprivileged 0 \
  --features nesting=1,mount=nfs \
  --onboot 0 \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.1.131/24,gw=192.168.1.1 \
  --rootfs local-lvm:8
```
3. Create slurm-node2 (Worker 2)
```
pct create 132 local:vztmpl/debian-12-standard_12.7-1_amd64.tar.zst \
  --hostname slurm-node2 \
  --memory 2048 --swap 1024 --cores 2 \
  --ostype debian --unprivileged 0 \
  --features nesting=1,mount=nfs \
  --onboot 0 \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.1.132/24,gw=192.168.1.1 \
  --rootfs local-lvm:8
```
Start all three containers
```
pct start 130 && pct start 131 && pct start 132
```
Step 1. Create an Ansible project directory on slurm-head.
```
mkdir -p /root/ansible-slurm/files
cd /root/ansible-slurm
```
Create the inventory file hosts.ini. You can easily add slurm-node3, slurm-node4, etc. there in the future.

Step 2. Copy the authoritative munge.key and slurm.conf from your running head node into the Ansible staging directory.
```
cp /etc/munge/munge.key /root/ansible-slurm/files/munge.key
cp /etc/slurm/slurm.conf /root/ansible-slurm/files/slurm.conf
```
Step 3. Create /root/ansible-slurm/provision_worker.yml.

Step 4. Install Ansible on slurm-head
```
apt install -y ansible
```
Execute the playbook against all worker nodes
```
cd /root/ansible-slurm
ansible-playbook -i hosts.ini provision_worker.yml
```
Step 5. Whenever you spin up an additional worker in the future (e.g. slurm-node3):
Create the container on Proxmox (pct create 133 ...).

Add its IP to hosts.ini.

Add the line NodeName=slurm-node3 ... to /etc/slurm/slurm.conf and update files/slurm.conf.

Run ansible-playbook -i hosts.ini provision_worker.yml.

On slurm-head, reload the cluster state and resume the node:
```
scontrol reconfigure
scontrol update NodeName=slurm-node3 State=RESUME
sinfo
```

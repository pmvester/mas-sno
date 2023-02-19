# mas-sno
Maximo Application Suite (Manage) on Single Node OpenShift
## Resources
- [Spin up a single-node Red Hat OpenShift cluster with one command](https://developer.ibm.com/tutorials/spin-up-a-single-node-openshift-cluster-with-one-command/)
## Installing SNO
```zsh
docker run -it --name sno --mount type=bind,source="$(pwd)",target=/root/sno pmv/fedora-sno:0.4
```
```zsh
git clone https://github.com/IBM/sno-on-ibm-cloud-vpc-ansible.git
```
In `ansible/roles/provision_kvm_host/defaults/main.yml` change `vsi_profile`from `bx2d-16x64` to `bx2d-32x128`.
In `ansible/roles/create_sno_vm/defaults/main.yml` change `sno_vm_vcpus`from `8` to `24` and `sno_vm_ram_mb` from `32768` to `98304`.
## Adding storage
Attach a new storage volume to your VSI.
![vsi-attach.png](/images/vsi-attach.png)
![vsi-storage.png](/images/vsi-storage.png)
```zsh
lsblk
```
```zsh
sudo fdisk /dev/vdf
```
```zsh
sudo mkfs.xfs /dev/vdf1
```
```zsh
sudo vi /etc/fstab
```
```zsh
sudo mkdir /data3tb
```
```zsh
sudo mount /data3tb
```
```zsh
sudo virsh pool-list --all
```
```zsh
sudo virsh pool-define-as data3tb dir - - - - "/data3tb"
```
```zsh
sudo virsh pool-start data3tb
```
```zsh
sudo virsh pool-autostart data3tb
```
```zsh
sudo virsh pool-info data3tb
```
![virsh pool-info](/images/virsh-pool-info.png)
```zsh
sudo virsh vol-create-as --pool data3tb --name data3tb.qcow2 --format qcow2 --capacity 2910G
```
```zsh
sudo virsh list
```
![virsh list](/images/virsh-list.png)
```zsh
sudo virsh attach-disk --domain alice-vm --source /data3tb/data3tb.qcow2 --subdriver qcow2 --target vdb
```
## Installing LVM
Create `namespace.yaml`.
```
apiVersion: v1
kind: Namespace
metadata:
  labels:
    openshift.io/cluster-monitoring: "true"
  name: openshift-storage
spec: {}
```
```zsh
oc login --token=sha256~zeavQvAC0epuUD729-zJgrDJnGqciqJDO2s6ySxu5ug --server=https://api.alice.snomas.cloud:6443
```
```zsh
oc create -f namespace.yaml
```
![lvm operator](/images/lvm-operator.png)
![lvm operator install 1](/images/lvm-operator-install-1.png)
![lvm operator install 2](/images/lvm-operator-install-2.png)
![lvm cluster create](/images/lvm-cluster-create.png)
```zsh
oc get pods -n openshift-storage
```
![get pods](/images/get-pods.png)
![storage classes](/images/storage-classes.png)

## Installing MAS
```zsh
docker run -it --name spectrum --mount type=bind,source="$(pwd)",target=/opt/app-root/src/masdir --rm quay.io/ibmmas/cli
```

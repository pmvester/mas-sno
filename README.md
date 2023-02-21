# mas-sno
Maximo Application Suite (Manage) on Single Node OpenShift. These are my personal notes and not intended to be a step by step instruction on how to perform an installation. However, the instructions contains enough information for a sufficiently experienced person to figure out how such an installation can be done.
## Resources
- [Spin up a single-node Red Hat OpenShift cluster with one command](https://developer.ibm.com/tutorials/spin-up-a-single-node-openshift-cluster-with-one-command/)
- [Add Storage to your KVM Linux Virtual Machine from the Command Line](https://www.youtube.com/watch?v=kjVXnhg92rw) (YouTube video)
- [ODF at the Edge: LVMO, MCG on SNO](https://www.youtube.com/watch?v=JcykZYeSI2Q) (YouTube video)
- [Creating a Directory-based Storage Pool with virsh](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/virtualization_administration_guide/sect-virtualization-storage_pools-creating-local_directories-virsh)
- [Configuring the registry for bare metal](https://docs.openshift.com/container-platform/4.10/registry/configuring_registry_storage/configuring-registry-storage-baremetal.html)
## Installing SNO
<!--
```zsh
docker run -it --name sno --mount type=bind,source="$(pwd)",target=/root/sno pmv/fedora-sno:0.4
```
-->
```zsh
git clone https://github.com/IBM/sno-on-ibm-cloud-vpc-ansible.git
```
In `ansible/roles/provision_kvm_host/defaults/main.yml` 
- change `vsi_profile`from `bx2d-16x64` to `bx2d-32x128`.

In `ansible/roles/create_sno_vm/defaults/main.yml` 
- change `sno_vm_vcpus`from `8` to `24`
- change `sno_vm_ram_mb` from `32768` to `98304`.

![group_vars/all](/images/group_vars-all.png)

Perform the steps described in [Spin up a single-node Red Hat OpenShift cluster with one command](https://developer.ibm.com/tutorials/spin-up-a-single-node-openshift-cluster-with-one-command/).

Update `/etc/hosts` with the values found in `auth/alice.hosts`.

## Adding storage
Attach a new storage volume to your VSI.
![vsi-attach.png](/images/vsi-attach.png)
![vsi-storage.png](/images/vsi-storage.png)
```zsh
lsblk
```
![lsblk](/images/lsblk.png)
```zsh
sudo fdisk /dev/vdf
```
![fdisk](/images/fdisk.png)
```zsh
sudo mkfs.xfs /dev/vdf1
```
```zsh
sudo mkdir /data3tb
```
```zsh
sudo vi /etc/fstab
```
add one line at the end of the file
```
/dev/vdf1 /data3tb xfs defaults 0 0
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
```zsh
oc login --token=<<token>> --server=https://api.alice.snomas.cloud:6443
```
Create a file named `namespace.yaml` with the following content.
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

## Configuring the registry

```zsh
oc patch config.imageregistry.operator.openshift.io/cluster --type=merge -p '{"spec":{"rolloutStrategy":"Recreate","replicas":1}}'
```
Create new 100G PVC type `File` named `image-registry-storage`.
```zsh
oc edit configs.imageregistry/cluster
```
Change `managementState`to `Managed` and update `storage` to:
```
storage:
  pvc:
    claim: image-registry-storage
```

## Installing MAS
```zsh
docker run -it --name mas --mount type=bind,source="$(pwd)",target=/opt/app-root/src/masdir --rm quay.io/ibmmas/cli
```
Update `/etc/hosts` with the values found in `auth/alice.hosts` from the Installing SNO step.
```zsh
mkdir ~/masdir
```
Copy your license file to `~/masdir/entitlement.lic`.
```
# AAA_ENTITLEMENT_KEY must be defined first.
export AAA_API_KEY=<<api key>>
export AAA_ENTITLEMENT_KEY=<<entitlement key>>
export CIS_APIKEY=$AAA_API_KEY
export CIS_CRN=<<crn key>>
export CIS_EMAIL=mikael.vester@se.ibm.com
export CIS_SUBDOMAIN=alice
export CPD_ENTITLEMENT_KEY=$AAA_ENTITLEMENT_KEY
export CPD_PRODUCT_VERSION=4.5.2
export DB2_BACKUP_STORAGE_ACCESSMODE=ReadWriteOnce
export DB2_BACKUP_STORAGE_CLASS=odf-lvm-vgmcg
export DB2_DATA_STORAGE_CLASS=odf-lvm-vgmcg
export DB2_INSTANCE_NAME=db2w-shared
export DB2_LOGS_STORAGE_CLASS=odf-lvm-vgmcg
export DB2_META_STORAGE_ACCESSMODE=ReadWriteOnce
export DB2_META_STORAGE_CLASS=odf-lvm-vgmcg
export DB2_TEMP_STORAGE_CLASS=odf-lvm-vgmcg
export DNS_PROVIDER=cis
export GRAFANA_INSTANCE_STORAGE_CLASS=odf-lvm-vgmcg
export IBMCLOUD_APIKEY=$AAA_API_KEY
export IBMCLOUD_RESOURCEGROUP=sno-rg
export IBM_ENTITLEMENT_KEY=$AAA_ENTITLEMENT_KEY
export MAS_APPWS_COMPONENTS="base=latest"
export MAS_APP_ID=manage
export MAS_APP_SETTINGS_DEMODATA=true
export MAS_CONFIG_DIR=/opt/app-root/src/masdir
export MAS_DOMAIN=alice.imomax.org
export MAS_ENTITLEMENT_KEY=$AAA_ENTITLEMENT_KEY
export MAS_INSTANCE_ID=mas8
export MAS_WORKSPACE_ID=masdev
export MONGODB_STORAGE_CLASS=odf-lvm-vgmcg
export PROMETHEUS_ALERTMGR_STORAGE_CLASS=odf-lvm-vgmcg
export PROMETHEUS_STORAGE_CLASS=odf-lvm-vgmcg
export SLS_ENTITLEMENT_KEY=$AAA_ENTITLEMENT_KEY
export SLS_LICENSE_FILE=~/masdir/entitlement.lic
export SLS_LICENSE_ID=46a7079eee48
export SLS_MONGODB_CFG_FILE=~/masdir/mongo-mongoce.yml
export SLS_STORAGE_CLASS=odf-lvm-vgmcg
export UDS_CONTACT_EMAIL=mikael.vester@se.ibm.com
export UDS_CONTACT_FIRSTNAME=Mikael
export UDS_CONTACT_LASTNAME=Vester
export UDS_STORAGE_CLASS=odf-lvm-vgmcg
export UPGRADE_IMAGE_REGISTRY_STORAGE=true
```
```zsh
ansible-playbook ibm.mas_devops.oneclick_core
```
```zsh
ansible-playbook ibm.mas_devops.oneclick_add_manage
```

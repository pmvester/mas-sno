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
```
# AAA_ENTITLEMENT_KEY must be defined first.
export AAA_API_KEY=1TwD69GN4IYIBTGFALTyrSTbisUjegYMHBCOln9de3_O
export AAA_ENTITLEMENT_KEY=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJJQk0gTWFya2V0cGxhY2UiLCJpYXQiOjE2NDkwNjc2NzUsImp0aSI6IjQ0NDQ2Nzk3YTM4YjRhMWE4NDM3NDI2Yzg1ZTZiODcxIn0.rd4TorEjW8E51Oa-KphN5Mf_T0AX8NK4VV76UREz04c
export CIS_APIKEY=$AAA_API_KEY
export CIS_CRN=crn:v1:bluemix:public:internet-svcs:global:a/1153702d2118844328b480b82517e235:1bdff21f-dc86-4f74-9b6b-0719eff09920::
export CIS_EMAIL=mikael.vester@se.ibm.com
export CIS_SUBDOMAIN=alice
export CPD_ENTITLEMENT_KEY=$AAA_ENTITLEMENT_KEY
export CPD_PRODUCT_VERSION=4.5.2
export DB2_INSTANCE_NAME=db2w-shared
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
docker run -it --name sno --mount type=bind,source="$(pwd)",target=/root/sno pmv/fedora-sno:0.4

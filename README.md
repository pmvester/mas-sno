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
## Installing LVM
## Installing MAS
```zsh
docker run -it --name spectrum --mount type=bind,source="$(pwd)",target=/opt/app-root/src/masdir --rm quay.io/ibmmas/cli
```

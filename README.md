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
## Adding storage
## Installing LVM
## Installing MAS
```zsh
docker run -it --name spectrum --mount type=bind,source="$(pwd)",target=/opt/app-root/src/masdir --rm quay.io/ibmmas/cli
```

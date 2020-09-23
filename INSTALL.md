# Install Cluster

Steps:

* Step 1: Running the pre-requisites playbook

```bash
ansible-playbook site-prerequisites.yml -i inventory --extra-vars "@custom_vars.yml"
```

* Step 2: Deploying the cluster

```bash
ansible-playbook site.yml -i inventory --extra-vars "@custom_vars.yml"
```

Notes:

* If you want to upgrade all centos packages, you need to use `-e upgrade_packages=true`. It will take around 30min to upgrade all the packages.
* If you have another partition to mount `/var/lib/docker`, you could to use `deploy_docker_lvm_storage=true`, the default disk is `/dev/sdb`

## Install ADDONS

### ISTIO

Istio will be installed by default, but if you prefer install it manually, you could run this playbook

```bash
sudo ansible-playbook site.yml -i inventory --extra-vars "@custom_vars.yml" -t istio -e "deploy_istio=true"
```

## Simple Cluster

```bash
ansible-playbook site-prerequisites.yml -i inventory --extra-vars "@custom_vars.yml" -e deploy_docker_lvm_storage=true 
ansible-playbook site.yml -i inventory --extra-vars "@custom_vars.yml"  -e "cluster_taint=false" -e "deploy_glusterfs=true"
```


## UNINSTALL

Steps:

```
ansible -m shell -a "systemctl stop kube-apiserver kube-controller-manager kube-scheduler" -i inventory master
ansible -m shell -a "systemctl stop etcd" -i inventory etcd
ansible -m shell -a "systemctl stop kubelet kube-proxy docker" -i inventory node

ansible -m shell -a 'rm -rf /var/lib/etcd/*' -i inventory etcd
ansible -m shell -a 'rm -rf /opt/kubernetes' -i inventory master
ansible -m shell -a 'rm -rf /var/lib/kubernetes /var/lib/kubelet /var/lib/kube-proxy' -i inventory node
```

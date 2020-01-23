# Install Cluster

Steps:

* Step 1: Running the pre-requisites playbook

```bash
ansible-playbook site-prerequisites.yml -i inventory-simple --extra-vars "@custom_vars.yml"
```

* Step 2: Deploying the cluster

```
ansible-playbook site.yml -i inventory-simple --extra-vars "@custom_vars.yml"
```

Notes:

* If you want to upgrade all centos packages, you need to use `-e upgrade_packages=true`. It will take around 30min to upgrade all the packages.
* If you have another partition to mount `/var/lib/docker`, you could to use `deploy_docker_lvm_storage=true`, the default disk is `/dev/sdb`

## Install ADDONS

### ISTIO

Istio will be installed by default, but if you prefer install it manually, you could run this playbook

```sh
sudo ansible-playbook site.yml -i inventory-lab --extra-vars "@custom_vars.yml" -t istio -e "deploy_istio=true"
```

## Simple Cluster

```shell
sudo ansible-playbook site-prerequisites.yml -i inventory-lab --extra-vars "@custom_vars.yml" -e deploy_docker_lvm_storage=true 
sudo ansible-playbook site.yml -i inventory-lab --extra-vars "@custom_vars.yml"  -e "cluster_taint=false" -e "deploy_glusterfs=true"
```


## UNINSTALL

Steps:

        sudo ansible -m shell -a "systemctl stop kube-apiserver kube-controller-manager kube-scheduler" -i inventory-lab master
        sudo ansible -m shell -a "systemctl stop etcd" -i inventory-lab etcd
        sudo ansible -m shell -a "systemctl stop kubelet kube-proxy docker" -i inventory-lab node

        sudo ansible -m shell -a 'rm -rf /var/lib/etcd/*' -i inventory-lab etcd
        sudo ansible -m shell -a 'rm -rf /opt/kubernetes' -i inventory-lab master
        sudo ansible -m shell -a 'rm -rf /var/lib/kubernetes /var/lib/kubelet /var/lib/kube-proxy' -i inventory-lab node

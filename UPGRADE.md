# UPGRADE

## UPGRADE KUBERNETES VERSION


### STEPS

Download the latest kubernetes version in our mirror server `https://mirror.enciso.website/repo/linux/kubernetes`. 

* To do that, run the following script into the server `dcbvm090pr011.enciso.website`

```sh
cd /public/repo/linux/kubernetes
./download-version.sh 1.17.1
```

* Into this playbook, modify the kubernetes version in this file `group_vars/all/main.yml`


```yaml
kubernetes_version: 1.17.1
``` 

* Run the playbook `site-upgrade.yml`

```sh
sudo ansible-playbook site-upgrade.yml -i inventory-tst -t upgrade_master -e upgrade_master=true --extra-vars "@custom_vars.yml" --limit=master
sudo ansible-playbook site-upgrade.yml -i inventory-tst -t upgrade_node -e upgrade_node=true --extra-vars "@custom_vars.yml" --limit=master
sudo ansible-playbook site-upgrade.yml -i inventory-tst -t upgrade_node -e upgrade_node=true --extra-vars "@custom_vars.yml" --limit=node_infra
sudo ansible-playbook site-upgrade.yml -i inventory-tst -t upgrade_node -e upgrade_node=true --extra-vars "@custom_vars.yml" --limit=node_compute
```

* Done. To test execute:

```sh
kubectl get nodes -o wide | grep 1.17.1
```

-----

## UPGRADE ETCD VERSION 

This process is not necessary, but if you want to do, please, be careful

### Steps

* Modify the kubernetes version into `groups/all/main.yml`

```yaml
etcd_version: 3.3.11
``` 

* Run the playbook `site-upgrade.yml`

```bash
sudo ansible-playbook site-upgrade.yml -i inventory-lab -t upgrade_etcd -e upgrade_etcd=true
```

-----

## UPGRADE DOCKER

Example: `node_infra`

```sh
sudo ansible -m shell -a "yum makecache; yum upgrade -y docker-ce" -i inventory-lab node_infra
sudo ansible-playbook site.yml -i inventory-lab -t docker-proxy --limit=node_infra
sudo ansible -m shell -a "systemctl restart kubelet" -i inventory-lab node_infra
```

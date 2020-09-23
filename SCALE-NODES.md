## SCALE-UP: ADD NEW NODES

Steps:

* In the inventory file, you need to create a group `new_nodes` with the nodes and their vars

```
[node:children]
new_nodes
...
...
[new_nodes]
kube-w03.mycompany.com  ipv4addr=10.64.13.220  k8s_node_group_name=compute  docker_lvm_setup=true  max_pods_per_node=220
```

* Run the ansible playbook

```
sudo ansible-playbook site-scaleup.yml -i inventory" \
-e "deploy_docker_lvm_storage=true" \
-e "deploy_workers=true" \
-e "deploy_calico=true"
```
> It is idempotent playbook, so you could repeat this process any times if in some step it give you an error

------

## SCALE-DOWN: DELETE NODES

To delete nodes from the cluster follow this steps

* Delete the node from the inventory file
* Drain all the containers from this node
* Finally delete the node

Ex:
```
kubectl drain kube-w03.mycompany.com --ignore-daemonsets
kubectl delete node kube-w03.mycompany.com
```

# KUBE-CLUSTER

<img src="https://kubernetes.io/images/kubernetes-horizontal-color.png" width="500" align="center">

This playbook is based in the documment [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)

<br/>

## Features

- Kubernetes 1.17
- High Availability Cluster
- Master nodes are running etcd daemon in cluster mode
- Automatic node register using bootstrap option
- RBAC Authentication
- Calico Network Support (new version)
- Docker 19.03
- CoreDNS 1.4
- GlusterFS - CNS
- Istio 1.4

## Requirements 

### Full cluster

* 13 VM's Centos 7

Setup:

| # | Type | CPU's | Memory | Disk |
|:--|:-----|:------|:-------|:-----|
| 2 | LoadBalancer | 1 vCPU | 1 GB  | SO: /dev/sda (30GB), Storage Container: /dev/sdb (50GB) |
| 3 | Etcd         | 4 vCPU | 4 GB  | SO: /dev/sda (30GB), Storage Container: /dev/sdb (50GB) |
| 3 | Master       | 4 vCPU | 8 GB  | SO: /dev/sda (30GB), Storage Container: /dev/sdb (50GB) |
| 3 | Node Infra   | 4 vCPU | 8 GB  | SO: /dev/sda (30GB), Storage Container: /dev/sdb (50GB), Glusterfs: /dev/sdc (150GB) |
| 2 | Node Compute | 4 vCPU | 8 GB  | SO: /dev/sda (30GB), Storage Container: /dev/sdb (50GB) |

### Simple cluster

* 1 master node
* 1 compute node


## INSTALLATION

Create a api domain to communicate with the API externally. Ex. `apik8s-lab.mycompany.com` 

To do that, you need to set the var `api_domain` in `group_vars/all/main.yml` file 

Then, follow the instructions detailed in [INSTALL.md](INSTALL.md).


## CLIENT SETUP 

Ex. env_site=lab

```
export USER=admin
export PASS=s3cr3t
export ENV_SITE=lab
kubectl config set-credentials $USER --username=$USER --password=$PASS
kubectl config set-cluster $ENV_SITE --server=https://apik8s-$ENV_SITE.mycompany.com --insecure-skip-tls-verify
kubectl config set-context k8s-$ENV_SITE --cluster=$ENV_SITE --user=$USER
kubectl config use-context k8s-$ENV_SITE
```

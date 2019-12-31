## KUBERNETES STORAGE

Here are the steps to deploy glusterfs into kubernetes. The example is on `PRD` environment.
First all, check if your 3 nodes have a disk storage like this `/dev/sdc`, the size could be 200GB

* Step 1: Checkout this projet into the master node

```
cd /opt
git clone https://github.com/gluster/gluster-kubernetes
```


create the namesapce

```
kubectl create ns glusterfs
```

* Step 2: Create a `topology.json` file into `/opt/gluster-kubernetes/deploy` with the follow content

```
{
  "clusters": [
    {
      "nodes": [
        {
          "node": {
            "hostnames": {
              "manage": [
                "dcbvm090pr347.domain.com"
              ],
              "storage": [
                "10.64.12.47"
              ]
            },
            "zone": 1
          },
          "devices": [
            "/dev/sdc"
          ]
        },
        {
          "node": {
            "hostnames": {
              "manage": [
                "dcbvm090pr348.domain.com"
              ],
              "storage": [
                "10.64.12.48"
              ]
            },
            "zone": 1
          },
          "devices": [
            "/dev/sdc"
          ]
        },
        {
          "node": {
            "hostnames": {
              "manage": [
                "dcbvm090pr349.domain.com"
              ],
              "storage": [
                "10.64.12.49"
              ]
            },
            "zone": 1
          },
          "devices": [
            "/dev/sdc"
          ]
        }
      ]
    }
  ]
}
```

* Step 3: Deploy the gluster and heketi 

```
cd /opt/gluster-kubernetes/deploy 
./gk-deploy -g -v -n glusterfs
```

* Step 4: Create the storage-class "standard,glusterfs-storage" and "default" 

Get the HEKETI_CLI_SERVER using this command

``` 
export HEKETI_CLI_SERVER=$(kubectl get svc/heketi --template 'http://{{.spec.clusterIP}}:{{(index .spec.ports 0).port}}' -n glusterfs)
```

Create the files:

```
cat <<EOF > /opt/gluster-kubernetes/deploy/gluster-storage-class.yaml
apiVersion: storage.k8s.io/v1beta1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: $HEKETI_CLI_SERVER
EOF
```

```
cat <<EOF > /opt/gluster-kubernetes/deploy/gluster-storage-class.yaml
apiVersion: storage.k8s.io/v1beta1
kind: StorageClass
metadata:
  name: glusterfs-storage
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: $HEKETI_CLI_SERVER
EOF
```

```
cat <<EOF > /opt/gluster-kubernetes/deploy/default-storage-class.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  name: default
parameters:
  resturl: $HEKETI_CLI_SERVER
provisioner: kubernetes.io/glusterfs
reclaimPolicy: Delete
EOF
```

```
cd /opt/gluster-kubernetes/deploy
kubectl create -f  standard-storage-class.yaml
kubectl create -f  glusterfs-storage-class.yaml
kubectl create -f  default-storage-class.yaml
```

Step 5: Testing provision with minio and helm

```
helm install --set serviceType=NodePort --name storage-test --namespace=default stable/minio
```
 
```
helm del --purge storage-test
```

## Uninstall 

```
kubectl delete deployments  -n glusterfs deploy-heketi
kubectl delete ds -n glusterfs glusterfs
kubectl delete secrets -n glusterfs heketi-config-secret
kubectl delete clusterrolebinding heketi-sa-view
kubectl delete sa heketi-service-account -n glusterfs

sudo ansible -m shell -a "rm -rf /var/lib/glusterd /etc/glusterfs /var/lib/heketi /var/log/glusterfs" -i inventory-lab node_infra
```

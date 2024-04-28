# magnum-capi

# Create prebuild image for k8s cluster
Today latest version of k8s supported is 1.27.x, for 1.28x, 1.29.x and 1.30.x, stil in develope

```
openstack image create ubuntu-2204-kube-v1.27.8 \
  --public \
  --disk-format=qcow2 \
  --container-format=bare \
  --property os_distro='ubuntu' \
  --file ubuntu-2204-kube-v1.27.8.qcow2
```

# Create template magnum for k8s
The template represent spesific version of k8s, some options that we can include in the template (the template contains default values) or can be changed when creating the cluster. The fully options can be check with command `openstack coe cluster template --help` if you don't have that command before, you can install in your venv with `pip install python-magnumclient`.

```
openstack coe cluster template create \
      --image ubuntu-2204-kube-v1.27.8 \
      --external-network public\
      --dns-nameserver 8.8.8.8 \
      --master-lb-enabled \
      --master-flavor m0-kubernetes \
      --flavor m0-kubernetes \
      --network-driver calico \
      --docker-storage-driver overlay2 \
      --coe kubernetes \
      --label kube_tag=v1.27.8 \
      k8s-v1.27.8
```

# Create k8s cluster
When creating the cluster we can changed default options from template, like internal network, subnet , flavor etc. 

```
openstack coe cluster create k8s-v1.27.8 --keypair sysadmin-key \
  --cluster-template k8s-v1.27.8  --fixed-subnet internal-subnet \
  --master-count 1 --node-count 1
```

# Create cluster with autoscalling turn on (default no)

```
openstack coe cluster create k8s-v1.27.8 --keypair sysadmin-key \
  --cluster-template k8s-v1.27.8 \
  --master-count 1 --node-count 1 \
  --labels auto_scaling_enabled=true,min_node_count=1,max_node_count=3 
```






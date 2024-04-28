# magnum-capi

1. [Create prebuild image for k8s cluster](#create-prebuild-image-for-k8s-cluster)
2. [Create template magnum for k8s](#create-template-magnum-for-k8s)
3. [Create k8s cluster](#create-cluster-with-autoscalling-turn-on-default-no)
4. [Create cluster with autoscalling turn on (default no)](#create-cluster-with-autoscalling-turn-on-default-no)
5. [Update cluster ](#update-cluster)
6. [Manual Scale Cluster]()
# 1. Create prebuild image for k8s cluster
Today latest version of k8s supported is 1.27.x, for 1.28x, 1.29.x and 1.30.x, stil in develope

```
openstack image create ubuntu-2204-kube-v1.27.8 \
  --public \
  --disk-format=qcow2 \
  --container-format=bare \
  --property os_distro='ubuntu' \
  --file ubuntu-2204-kube-v1.27.8.qcow2
```

# 2. Create template magnum for k8s
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
      template-k8s-v1.27.8
```

# 3. Create k8s cluster
When creating the cluster we can changed default options from template, like internal network, subnet , flavor etc. 

```
openstack coe cluster create mycluster --keypair sysadmin-key \
  --cluster-template k8s-v1.27.8  --fixed-subnet internal-subnet \
  --master-count 1 --node-count 1
```

# 4. Create cluster with autoscalling turn on (default no)
[How autoscalling work ?](https://github.com/pahrialms/magnum-capi/blob/main/autoscalling/autoscalling_flow.md)
```
openstack coe cluster create mycluster --keypair sysadmin-key \
  --cluster-template k8s-v1.27.8 \
  --master-count 1 --node-count 1 --fixed-subnet internal-subnet \
  --labels auto_scaling_enabled=true,min_node_count=1,max_node_count=3 
```

# 5. Update cluster 
A cluster can be modified using the ‘cluster-update’ command, for example:
```
openstack coe cluster update mycluster replace node_count=8
```
The parameters are positional and their definition and usage are as follows.

`<cluster>`
  
This is the first parameter, specifying the UUID or name of the cluster to update.

`<op>`
  
This is the second parameter, specifying the desired change to be made to the cluster attributes. The allowed changes are ‘add’, ‘replace’ and ‘remove’.

`<attribute=value>`

This is the third parameter, specifying the targeted attributes in the cluster as a list separated by blank space. To add or replace an attribute, you need to specify the value for the attribute. To remove an attribute, you only need to specify the name of the attribute. Currently the only attribute that can be replaced or removed is ‘node_count’. The attributes ‘name’, ‘master_count’ and ‘discovery_url’ cannot be replaced or delete. The table below summarizes the possible change to a cluster.  
![image](https://github.com/pahrialms/magnum-capi/assets/82088448/3b22996e-dbad-4124-9a76-ea43bae951a8)

# Manual Scale Cluster
Scaling a cluster means adding servers to or removing servers from the cluster. Currently, this is done through the ‘cluster-update’ operation by modifying the node-count attribute, for example:

```
openstack coe cluster update mycluster replace node_count=8
```

# Delete
The ‘cluster-delete’ operation removes the cluster by deleting all resources such as servers, network, storage; for example:
```
openstack coe cluster delete mycluster
```
The only parameter for the cluster-delete command is the ID or name of the cluster to delete. Multiple clusters can be specified, separated by a blank space.

# Monitoring stack

Monitoring use ceilometer,aodh,gnocchi (optional use grafana with datasource gnocchi)

We can list the metrics in our project using the command:
```
openstack metric list
```

To view more information about the metric, we can use:
```
openstack metric show <uuid>
```

We can view the metric resource list as well. The following command will return a table containing every single resource (instance, instance disk, etc) which metrics are attached to.
```
openstack metric resource show <resource-id>
```

To view the measurements of a metric, we can use the command:
```
openstack metric measures show <metric-id>
```
To view the resource which has a metric linked to it, we can use the command:
```
openstack metric resource show <resource-id>
```



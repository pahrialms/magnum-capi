# Table of Content magnum-cluster-api

References : 
- https://vexxhost.github.io/magnum-cluster-api/
- https://docs.openstack.org/magnum/latest/user/index.html

1. [Create prebuild image for k8s cluster](#1-create-prebuild-image-for-k8s-cluster)
2. [Create template magnum for k8s](#2-create-template-magnum-for-k8s)
3. [Create k8s cluster](#3-create-k8s-cluster)
4. [Create cluster with autoscalling turn on (default no)](#4-create-cluster-with-autoscalling-turn-on-default-no)
5. [Get Kubeconfig](#5-get-kubeconfig)
6. [Manual Scale Cluster](#6-manual-scale-cluster)
7. [Delete cluster](#7-delete-cluster)
8. [Monitoring stack](#8-monitoring-stack-cluster)
9. [Upgrade cluster](#9-upgrade-cluster)
10. [Monitoring container](#10-monitoring-container)


# 1. Create prebuild image for k8s cluster

Today latest version of k8s supported is 1.27.x, for 1.28x, 1.29.x and 1.30.x, stil on developing

```
openstack image create ubuntu-2204-kube-v1.26.11 \
  --public \
  --disk-format=qcow2 \
  --container-format=bare \
  --property os_distro='ubuntu' \
  --file ubuntu-2204-kube-v1.26.11.qcow2
```

# 2. Create template magnum for k8s
The template represent spesific version of k8s, some options that we can include in the template (the template contains default values) or can be changed when creating the cluster. The fully options can be check with command `openstack coe cluster template --help` if you don't have that command before, you can install in your venv with `pip install python-magnumclient`.

```
openstack coe cluster template create \
      --image ubuntu-2204-kube-v1.26.11 \
      --external-network public\
      --dns-nameserver 8.8.8.8 \
      --master-lb-enabled \
      --master-flavor m0-kubernetes \
      --flavor m0-kubernetes \
      --network-driver calico \
      --docker-storage-driver overlay2 \
      --coe kubernetes \
      --label kube_tag=v1.26.11 \
      template-k8s-v1.26.11
```

# 3. Create k8s cluster
When creating the cluster we can changed default options from template, like internal network, subnet , flavor etc. Full command `openstack coe cluster --help`. 
* note :
  - Flavor for master and worker can be changed with the options `--master-flavor` for master and `--flavor` for worker.
  - The number of master nodes `--master-count` can only be an odd number.
  - When enabling autoscalling it is also necessary to add the options `min_node_count` and `max_node_count`
  - [How autoscalling work ?](https://github.com/pahrialms/magnum-capi/blob/main/autoscalling/autoscalling_flow.md)
  - When using persistent storage need to specify label `boot_volume_type` and `boot_volume_size`

a. Create cluster k8s with autoscaling enabled and ephemeral storage

```
openstack coe cluster create mycluster \
    --keypair sysadmin-key \
    --cluster-template k8s-v1.27.8 \
    --master-count 1 \
    --node-count 1 \
    --merge-labels \
    --labels kube_tag=v1.27.8,auto_scaling_enabled=true,min_node_count=1,max_node_count=3,octavia_provider=ovn
```

b. Create cluster k8s with autoscaling enabled and persistent storage

```
openstack coe cluster create mycluster \
    --keypair sysadmin-key \
    --cluster-template k8s-v1.27.8 \
    --master-count 1 \
    --node-count 1 \
    --merge-labels \
    --labels kube_tag=v1.27.8,auto_scaling_enabled=true,min_node_count=1,max_node_count=3,octavia_provider=ovn,boot_volume_type=VT1,boot_volume_size=20
```

# 5. Get Kubeconfig

```
openstack coe cluster config <cluster-name>
# sample
openstack coe cluster config mycluster
```

# 6. Manual scale cluster
Scaling a cluster means adding servers to or removing servers from the cluster. for example:

```
openstack coe nodegroup create mycluster test-ng --node-count 1 --role test
```

# 7. Delete cluster
The ‘cluster-delete’ operation removes the cluster by deleting all resources such as servers, network, storage; for example:
```
openstack coe cluster delete mycluster
```
The only parameter for the cluster-delete command is the ID or name of the cluster to delete. Multiple clusters can be specified, separated by a blank space.

# 8. Monitoring stack cluster

Monitoring or reading cpu, ram and other metrics is still the same as a normal vm, using ceilometer, aodh, gnocchi (optionally using graphana with gnocchi datasource)

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
# 9. Upgrade cluster

In order to upgrade a cluster, you must have a cluster template pointing at the image for the new Kubernetes version and the kube_tag label must be updated to point at the new Kubernetes version. We can't skip major versions when upgrading, it must be per version, so if the k8s version is 1.25.x, it cannot be directly upgraded to 1.27.x, it must go to 1.26.x first and then to 1.27.x.
```
openstack coe cluster upgrade mycluster k8s-v1.27.8
```

# 10. Monitoring container

We can add additional container monitoring with helm and also we can install kubernetes dashboard



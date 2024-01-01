# Overview

kubespray testing

## kubespray-2.23.1

```bash
# kubespray has specific package version requirements
# python 3.12 is no good for rumel.yaml.clib
# python 3.11
python3.11 -m venv venv
. venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
pip install ansible-core==2.14.11
```

```bash
# Per README.md
cp -Rvfp inventory/sample inventory/kh1
# Edit inventory/kh1/inventory.ini to add host names

# declare current ips for kh1n1 kh1n2 kh1n3 kh1n4 (must be ips)
declare -a IPS=(192.168.1.160 192.168.1.162 192.168.1.166 192.168.1.165)
# Creates inventory/kh1/group_vars
CONFIG_FILE=inventory/kh1/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

# Check inventory/kh1/group_vars
# inventory/kh1/group_vars/all/all.yml
Nothing changed

# inventory/kh1/group_vars/k8s_cluster/k8s-cluster.yml
Nothing changed
kube_version: v1.27.7
kube_network_plugin: calico
kube_service_addresses: 10.233.0.0/18
kube_pods_subnet: 10.233.64.0/18
kube_network_node_prefix: 24
*cluster_name: kh1.local

# Reset (not run on first install)
ansible-playbook -i inventory/kh1/hosts.yaml  --become --become-user=root reset.yml

# Install
# Takes about 20 min to run
# Connects by ip address (not hostname)
ansible-playbook -i inventory/kh1/hosts.yaml  --become --become-user=root cluster.yml
reset && ansible-playbook -i inventory/kh1/hosts.yaml -u localsysadmin -b -k -K cluster.yml

```

## inventory.py output

```text
bluefield@tvatower ~/github-einwolf/kubernetes-kubespray-setup/kubespray-2.23.1$ declare -a IPS=(192.168.1.160 192.168.1.162 192.168.1.166 192.168.1.165)

bluefield@tvatower ~/github-einwolf/kubernetes-kubespray-setup/kubespray-2.23.1$ CONFIG_FILE=inventory/kh1/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
DEBUG: Adding group all
DEBUG: Adding group kube_control_plane
DEBUG: Adding group kube_node
DEBUG: Adding group etcd
DEBUG: Adding group k8s_cluster
DEBUG: Adding group calico_rr
DEBUG: adding host node1 to group all
DEBUG: adding host node2 to group all
DEBUG: adding host node3 to group all
DEBUG: adding host node4 to group all
DEBUG: adding host node1 to group etcd
DEBUG: adding host node2 to group etcd
DEBUG: adding host node3 to group etcd
DEBUG: adding host node1 to group kube_control_plane
DEBUG: adding host node2 to group kube_control_plane
DEBUG: adding host node1 to group kube_node
DEBUG: adding host node2 to group kube_node
DEBUG: adding host node3 to group kube_node
DEBUG: adding host node4 to group kube_node
```

## Groups check error

CVE bugfix
https://github.com/kubernetes-sigs/kubespray/issues/10688

`pip install ansible-core==2.14.11`

```
fatal: [node1]: FAILED! => {"msg": "The conditional check 'groups.get('kube_control_plane')' failed. The error was: Conditional is marked as unsafe, and cannot be evaluated."}
```

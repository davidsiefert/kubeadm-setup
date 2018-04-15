# kubeadm-setup
Notes for setting up a kubernetes cluster using `kubeadm`

## Installing `kubeadm`

### https://kubernetes.io/docs/setup/independent/install-kubeadm/

MAC addresses must be unique for each node:
```
ifconfig -a
```

Product UUID must be unique for each node:
```
sudo cat /sys/class/dmi/id/product_uuid
```

These two pieces of information are used to provide identity to each node automagically.

The following default port assignments are used by kubernetes:

* Master: 6443, 2379, 2380, 10250, 10251, 10252, 10255
* Node: 10250, 10255, 30000-32767

Install docker:

```
apt-get update
apt-get install -y apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable"

apt-get update
apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}')

```

Install `kubeadm`, `kubelet`, `kubectl`:
```
apt-get update
apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt-get update
apt-get install -y kubelet kubeadm kubectl
```

Your docker cgroup driver must match your kubeadm config:
```
docker info | grep -i cgroup
cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

At the moment, kubelet is not even configured to use cgroup... maybe it defaults to cgroupfs?
Current status of kubelet is that it failed to start because `/etc/kubernetes/pki/ca.crt` does not exist.

We move on...

# https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/

Initialize the master:
```
kubeadm init
```

This fails out of the gate due to preflight checks:

1. WARNING: crictl not found in system path
2. ERROR: running with swap on is not supported, disable swap


# prepare
ssh to master node 

Become root, update and upgrade system

`apt-get update && apt-get upgrade -y`

Install an editor like vim if there needed.

`apt-get install -y vim`

# Install container runtime

Take docker as an example

`apt-get install docker.io`

# Add a new repo for kubernetes

Create a the file and add an entry for the main repo

`vim /etc/apt/sources.list.d/kubernetes.list`

Add the following 4 sections into the file

`deb http://apt.kubernetes.io/ kubernetes-xenial main`

Add a GPG key for the packages

`curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -`

Update with the new repo declared
`apt-get update`

# Install the software

Install kubectl, kubeadm and kubelet, determine the version

`apt-get install -y kubeadm=1.18.1-00 kubelet=1.18.1-00 kubectl=1.18.1-00`

Hold the software

`apt-mark hold kubelet kubeadm kubectl`

# Install pod network

Take calico as an example. Download calido config file.

`wget https://docs.projectcalico.org/manifests/calico.yaml`

Go through the file and notice the ipv4 pool

`less calico.yaml`

```
# The default IPv4 pool to create on startup if none exists. Pod IPs will be
# chosen from this range. Changing this value after installation will have
# no effect. This should fall within`--cluster-cidr`.
         - name: CALICO_IPV4POOL_CIDR
           value:"192.168.0.0/16"
```

Check ip address of master server

`ip addr show`

Add an local DNS in hosts file, open the hosts file

`vim /etc/hosts`

Add a line:

`ip k8smaster`

# Bootstrap a cluster

Create a configuration file for the cluster

`vim kubeadm-config.yaml`

Add the following content

```
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: 1.18.1
controlPlaneEndpoint:"k8smaster:6443"
networking:
  podSubnet: 192.168.0.0
```

Initialize the master

`kubeadm init --config=kubeadm-config.yaml --upload-certs | tee kubeadm-init.out`

Allow non-root user to access the cluster


`exit`

`mkdir -p $HOME/.kube`

`sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`

`sudo chown $(id -u):$(id -g) $HOME/.kube/config`

Apply pod network plugin to the cluster

`sudo cp /root/calico.yaml .`

`kubectl apply -f calico.yaml`


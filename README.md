Configure a 4 node k8s with Vagrant and libvirt VM provider

This guide helps to setup a 4 node k8s cluster using k3s and also provides guidelines to attach and detach virtual block devices

### Pre-requisites

This guide expects the following components to be installed

#### KVM and its utilities :-

```sh
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
```

Verify the installation using 

```sh
virsh list --all
```

#### Start libvirtd service

```sh
sudo systemctl enable --now libvirtd
```

Verify by checking the status

```sh
sudo systemctl status libvirtd
```

#### Install virt-manager

```sh
sudo apt install virt-manager
```

Verify by starting the GUI

```sh
sudo virt-manager
```

#### Install vagrant

```sh
 wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg

 echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

 sudo apt update && sudo apt install vagrant
```


### Spin up VMs using vagrant

#### Clone the repository

```sh
git clone 
cd vagrant
```

#### Spin up VMs using vagrant

```sh
vagrant up
```

Verify the VMs by

```sh
vagrant global-status
```

the master and the 4 nodes should be in `running` state

For eg,

```sh
id       name   provider state   directory                                     
-------------------------------------------------------------------------------
17d3cdf  node1  libvirt running /home/praveen/go/src/github.com/minio/vagrant 
3f24371  node3  libvirt running /home/praveen/go/src/github.com/minio/vagrant 
7bf13d4  node4  libvirt running /home/praveen/go/src/github.com/minio/vagrant 
2ba66d5  node2  libvirt running /home/praveen/go/src/github.com/minio/vagrant 
204f6b1  master libvirt running /home/praveen/go/src/github.com/minio/vagrant 
```

### Setup k8s on the master and nodes

- Setup k3s on master

#### SSH into the master node

```sh
vagrant ssh master
```

#### Setup k3s

```sh
curl -sfL https://get.k3s.io | sh -
```

#### Save the kubeconfig

```sh
cat /etc/rancher/k3s/k3s.yaml
```

#### Copy the k3s token from the master node (to be used later)

```sh
cat /var/lib/rancher/k3s/server/node-token
```

#### Make a note of the master's ip

```sh
ip a s
```

### Setup k3s on worker nodes (to be executed on each worker node)

```sh
curl -sfL https://get.k3s.io | K3S_URL=https://<ip_of_the_master_node>:6443 K3S_TOKEN=<copied_token_from_master> sh -
```

#### Add the following host entries in `/etc/hosts/`

```sh
100.0.0.1 master.k8s.com master
100.0.0.2 node1.k8s.com node1
100.0.0.3 node2.k8s.com node2
100.0.0.4 node3.k8s.com node3
100.0.0.5 node4.k8s.com node4
```

(the master can also update the host entries accordingly)

### Check the cluster's health

#### Configure the kubectl in your client/host machine pointing to the cluster

Eg,

```sh
export KUBECONFIG=k3s.yaml
```

#### Check the status of the nodes by

```sh
kubectl get nodes
```

### Add LVs to the nodes

The following script will create one LV of size 800MiB. Feel free to customize as per your needs.

```sh
sudo truncate --size=1G /tmp/disk-1.img        
sudo losetup --find /tmp/disk-1.img
device=($(sudo losetup --noheadings --output NAME --associated /tmp/disk-1.img))
sudo pvcreate "${device}"
sudo vgcreate "vg-0" "${device}"
sudo lvcreate --name=lv-0 --size=800MiB vg-0
```

To remove the LVs,

```sh
sudo lvremove vg0
sudo vgremove vg0
sudo pvremove <loop1> <loopN>..
sudo losetup --detach-all
```

### Add virtual block devices using `virsh`

The following steps will attach a block device of size 1G to the respective node

STEP 1: Create a raw image using `qemu-img`

```sh
qemu-img create -f raw "vdb.img" 1G
```

STEP 2: Create a config file (.xml)

```xml
<disk type='file' device='disk'>
  <source file='vdb.img'/>
  <target dev='vdb'/>
  <serial>node1vdbserial</serial>
</disk>
```

(Note: Save the file in .xml)

STEP 3: Attach the device using `virsh` in node1

```sh
sudo virsh attach-device vagrant_node1 <file>.xml --live --persistent
```

STEP 4: SSH into the node

```sh
lsblk
```

Check if a new block device has been added


To detach the device,

```sh
sudo virsh detach-device vagrant_node1 <file>.xml
```

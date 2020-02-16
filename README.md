# Overview

We're running all MVP Studio projects in Kubernetes on [Rancher](https://rancher.com/docs/rancher/v2.x/en/overview/). This
repository contains non-project specific configuration scripts, Kubernetes configuration files, etc. that allow you to
provision a cluster to work with the rest of the MVP Studio system.

## Accessing Rancher and Setting up Kubectl

The master node is also running the main rancher container:
```
https://172.19.250.22:8443/
```

The credentials to access Rancher can be found in the standard MVP Studio secrets bucket (`passwords.mvpstudio.org`
bucket on GCE). Once in Rancher you can click the `Kubeconfig file` link to get a `~/.kube/config` file.

## Directories

The subdirectories in here are:

* bootstrap: these are configurations and scripts that need to be run exactly once when a new cluster is created.
* running: anything that is running in our production cluster belongs here. That is, if you haven't changed anything
  then running `kubectl apply -f .` from this directory shouldn't change anything.

**Never** modify cluster state or resources manually: that makes it very hard to recreate the cluster state for a new
cluster and leaves no audit trail. Instead, add, modify, or delete files here. If done correctly we should be able to
exactly re-create the state of our cluster (in another region for example in case of an outage) but `kubectl apply -f .`
in the `boostrap` directory followed by `kubectl apply -f .` in the `running` directory.

### Ambassador

All of our cluster routing is handled by [Ambassador](https://www.getambassador.io/). If you like to debug the Amassador
routing you must first find a running ambassador pod like:

```
$ kubectl get --namespace=ambassador pods
NAME                         READY   STATUS    RESTARTS   AGE
ambassador-8db4cb58d-gsmjj   1/1     Running   0          34d
ambassador-8db4cb58d-t9q2b   1/1     Running   0          34d
ambassador-8db4cb58d-tfqnn   1/1     Running   0          34d
```

Obviously the actual pod names will likely be different when you run this. Once you have the list of pods select one and
set up port forwarding:

```
$ kubectl --namespace=ambassador port-forward ambassador-8db4cb58d-gsmjj 8877 8877
```

and you can then point your browser to `http://localhost:8877/ambassador/v0/diag/` to see diagnostics.


## Hardware Configuration

#### 4 physical blades
- Debian 10
- 2 Nic cards 
- 2 cpus (8 cores each)
- 8gb RAM
- mirrored 300gb hdd (raid 1)


#### San
- 4-6 TB (Raid 10)


#### Node ips
172.19.250.20
172.19.250.22 (master)
172.19.250.24
172.19.250.26

#### External static ip
50.201.1.196 -- this is the VPN and should only be used for the VPN
50.201.1.197 -- this is the HAProxy instance and all traffic that hits this will go to Ambassador

#### users
- mvp (sudoer)


## Node Setup

In the future it'd be create to create an ansible workflow for this.


### The following tasks must be done on each node. 

#### Install Sudo

```
apt-get install sudo
```

#### Create Users

NOTE: So far Continuu folks have been creating the mvp user and adding it to the sudo group.

as root
```
adduser mvp
usermod -aG sudo mvp
```

as mvp (will need to logout for the new group to take effect)
```
sudo groupadd docker
sudo usermod -aG docker mvp
sudo mkdir ~
sudo chown mvp ~
mkdir ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
```

### Add public ssh key to each host

```
cat ~/.ssh/id_rsa.pub
pbcopy < ~/.ssh/id_rsa.pub # copies to clipboard on a mac
```
```
echo <public key> > ~/.ssh/authorized_keys
```

#### Configure SSH

sudo vi /etc/ssh/sshd_config
```
AllowTcpForwarding yes
```

```
sudo service sshd restart
```

#### Install Docker
```
sudo apt-get update
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

#### Set iptables to legacy mode

This is due to debuian buster using iptables v1.8.x which uses nftables as the underlying implementation, and kubernetes is not currently compatible with nftables.

[See this issue](https://github.com/kubernetes/kubernetes/issues/71305)

```
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
```

#### Install rancher on the master node

Rancher uses its own etcd database for storage, we are volume mounting `/var/lib/rancher` to `/opt/rancher` on the host to persist that database.

```
docker run -d --name rancher --restart=unless-stopped -v /opt/rancher:/var/lib/rancher -p 8080:80 -p 8443:443 rancher/rancher
```

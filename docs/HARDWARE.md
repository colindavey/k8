# Hardware Configuration

As noted elsewhere, some of the hardware configuration is now handled by [our Ansible scripts](../ansible/README.md).
Going forward this is preferred. However, the initial setup of the nodes was done manually following the steps below.

## 4 physical blades
- Debian 10
- 2 Nic cards 
- 2 cpus (8 cores each)
- 8gb RAM
- mirrored 300gb hdd (raid 1)


## San
- 4-6 TB (Raid 10)

See the [storage documentation](./STORAGE.md) for more details on how we've mounted and used this SAN array in the
cluster.


#### Node ips
* 172.19.250.20
* 172.19.250.22 (master)
* 172.19.250.24
* 172.19.250.26

#### External static ip
* 50.201.1.196 -- this is the VPN and should only be used for the VPN
* 50.201.1.197 -- this is the HAProxy instance and all traffic that hits this will go to Ambassador

#### users
- mvp (sudoer)


## Node Setup

The initial node setup was done manually. The following documents the steps that were performed. However, the storage
was configured via [Ansible](https://docs.ansible.com/) and we'd like to eventually migrate the steps documented below
to that Ansible script. The current Ansible scripts can be found [in the ansible directory](../ansible/README.md).


# The following tasks must be done on each node. 

#### Install Sudo

```
apt-get install sudo
```

#### Install Python

This is needed by Ansible.

```
apt-get install python
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

This is due to debian buster using iptables v1.8.x which uses nftables as the underlying implementation, and kubernetes
is not currently compatible with nftables.

[See this issue](https://github.com/kubernetes/kubernetes/issues/71305)

```
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
```

#### Install rancher on the master node

Rancher uses its own etcd database for storage, we are volume mounting `/var/lib/rancher` to `/opt/rancher` on the host
to persist that database.

```
docker run -d --name rancher --restart=unless-stopped -v /opt/rancher:/var/lib/rancher -p 8080:80 -p 8443:443 rancher/rancher
```

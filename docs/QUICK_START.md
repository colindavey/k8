# Quick Start

To get started both the new developer and some existing MVP Studio admins must do a few things. Each step below has
either a `(DEV)` or `(ADMIN)` after it indicating whom has to perform the task. The steps are as follows:

1. Install and set up [WireGuard](https://www.wireguard.com/). (DEV)
2. Grant new user access to the WireGuard VPN. (ADMIN)
3. Add new user to the ["K8 Users" group in the MVP Studio Github
   organization](https://github.com/orgs/MVPStudio/teams/k8s-users). (ADMIN)
4. Configure WireGuard to access the MVP Studio cluster. (DEV)
5. Connect to the VPN and sign into Rancher with your GitHub credentials. (DEV)
6. Download k8's configuration file from Rancher. (DEV)
7. Grant developer permissions to one or more projects/namespaces. (ADMIN)
8. Verify setup. (DEV)

Note that these are somewhat ordered. The new developer must install WireGuard to generate a private/public key pair
before they can be added to the VPN, and the admin must add the user to the "K8 Users" github group before the developer
can log into Rancher.

Each step is detailed below.

# Set Up WireGuard

To access the [WireGuard](https://www.wireguard.com/) VPN users must first [install
WireGuard](https://www.wireguard.com/install/). Users then use WireGuard to generate a public and a private key. Keys
can be generated [as per the docs](https://www.wireguard.com/quickstart/#key-generation) though Windows and OSX clients
also have GUI ways to do this.

Once you have a public/private key pair send _only the public key_ to the admin who is helping to get you set up on the
VPN.

# Grant Access to WireGuard VPN

Edit [the `ansible/template/wg0.conf` config file](../ansible/templates/wg0.conf) adding a new `[Peer]` section adding
the user's public key. Each user must be given an IP address in the `100.65.0.XXX/10` range that is _unique_ across all
other MVP Studio users. We keep the `wg0.conf` file sorted by IP address so you can just look at the last address
assigned in that file and assign the next one in sequence.  Be sure to add a comment indicating what actual human the
public key belongs to. Finally, to apply the changed config use Ansible:

```
ansible-playbook -i inventory_wg.yml -l master --tags=wg_config --ask-vault-pass playbook.yml
```

You can find the password for the Ansible vault in the `continu-k8.txt` file in our secrets bucket
(`passwords.mvpstudio.org` bucket on GCE).

# Add To K8 Users Group

Go to the [K8 Users group](https://github.com/orgs/MVPStudio/teams/k8s-users) and add the user to this group. Rancher is
set up to only allow access to people in this group. Note that adding a user to this group grants access to Rancher but
not to any clusters, projects, or namespaces.

# Configure WireGuard

The admin who is helping to set you up will tell you the unique IP address you have been assigned for the VPN. You must
then set up a WireGuard configuration file like the following:

```
[Interface]
Address = 100.65.0.XXX/10
PrivateKey = <private key here>
ListenPort = 51820

[Peer]
Endpoint = 50.201.1.196:51820
PublicKey = zEEB7K7+iY4OOZTYffQI7s0xsC2bq6aO+B6RTs6BzVo=
AllowedIPs = 100.64.0.1/32
PersistentKeepalive = 25
```

The `[Peer]` section should be copied unaltered. The `[Interface]` section is mostly as shown above except that the
`Address` should be the address assigned to you by the admin and the `PrivateKey` should be your unique private key
(remember not to share your private key with anyone).

# Connect to the VPN and Sign into Rancher

Connect to the VPN. The exact steps differ from OS to OS but Google something like "connect to WireGuard on OSX" or
similar should give you the information you need. Once on the VPN visit `https://100.64.0.1:8443/` in the browser of
your choice. We haven't bother setting up SSL since this is behind our VPN so accept whatever warning your browser shows
about the missing or invalid certificate. You will be presented with a login page. Click the "login with github" button
and login using your github account. You will be presented with a page listing clusters; select `mvp-studio` and click
the `Kubeconfig file` button and download or copy that file to `~/.kube/config`. At this point you won't have hardly any
permissions but you should be able to run `kubectl cluster-info` and see something like:

```
Kubernetes control plane is running at https://100.64.0.1:8443/k8s/clusters/c-pv74p
CoreDNS is running at https://100.64.0.1:8443/k8s/clusters/c-pv74p/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

TODO: I'm actually not sure we've even granted permissions to `kubectl cluster-info` that might fail. If it doesn't
delete this TODO. If it does, lets find another quick way to verify setup.

# Grant Permissions

Using the Rancher GUI click on `Projects/Namespaces`, find the project or projects the new developer will work on and
add them to the project.

# Verify Access

Finally, if everything has been set up correctly you should be able to run:

```
kubectl get -n <namespace> pods
```

where `<namespace>` is one of the namespaces to which you were granted access. The output of that command might be empty
if there isn't anything yet running in your namespace; the key thing is that you don't get an error message.

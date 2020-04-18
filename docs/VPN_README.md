# VPNs Plural

We have 2 VPNs. The first is an [OpenVpn](https://openvpn.net/) created by Continu for us. This gives us direct access
to all the machines in the cluster and we can ssh to individual nodes, etc. This is nice but it has a few downsides:

* Adding additional people to the VPN requires talking to Continu to set up certs and things.
* It provides a lot of access including the ability to send packets to individual nodes!

What we really want is the above VPN for MVP Studio admins but for regular users who just need to run `kubectl` we want
a VPN that limits their access to just the master server and we'd like to be able to add and remove new users easily and
without help from Continu. To this end we also have a simple [WireGuard](https://www.wireguard.com/) VPN.

# The WireGuard VPN

To access the [WireGuard](https://www.wireguard.com/) VPN users must first install WireGuard. To add a new user we then
run the [add_user.py](../wireguard/add_user.py) script. That script spits out a WireGuard configuration file that can be
imported into WireGuard (how exactly depends on the OS and which version you installed but it's pretty obvious - on
Linux it can be used via `wg-quick up ./wg-mvp.conf`). It also updates the [WireGuard configuration Jinja 2
template](../ansible/templates/wg0.conf) which can then be applied to the master server by somebody already on the VPN
via:

```
ansible-playbook -i inventory.yml -l master --ask-vault-pass playbook.yml
```

Note that our playbook assumes you're on the OpenVpn not the WireGuard one. If you want to run it while on the WireGuard
VPN you need to change the following in inventory.yml:

```
      master:
         hosts:
            master1:
               ansible_host: 172.19.250.22
```

to:

```
      master:
         hosts:
            master1:
               ansible_host: 100.64.0.1
```

to use the WireGuard IP Address for the server. Finally, create a pull request to check in the modified `wg0.conf` file.

The master server now has IP address 100.64.0.1 on the WireGuard network.

# The OpenVpn VPN

To connect to the "full VPN" that gives you access to not only the master server but also all the minions you will need
Continu to set up a username, password, and a variety of certificates and keys. They will general provide all this
information to you on a thumb drive or similar. Once you have the drive you'll need to configure some
[OpenVpn](https://openvpn.net/) compatible client.

## Linux

On Linux the easiest setup is probably to use the [Network
Manager](https://wiki.archlinux.org/index.php/NetworkManager). That package provides several ways to configure a
connection. The `nm-connection-editor` command will bring up a GUI to let you edit a configuration and most desktop
provide tray applets to configure and connect to various networks, including VPNs. On Debian Buster an `apt-get install
network-manager-openvpn-gnome` was sufficient to install the OpenVpn extension to the network manager and update the
tray applets to enable quick, GUI connections to the VPN.

However, you must first set up the VPN connection. The easiest way to do this is probably to first copy and edit the
following `.ovpn` file:

```
# Change the following 4 paths to point to the appropriate files
ca '/home/oliver/Documents/MVPStudio/vpn/certs/ca.crt'
cert '/home/oliver/Documents/MVPStudio/vpn/certs/cert.crt'
key '/home/oliver/Documents/MVPStudio/vpn/certs/key.key'
tls-auth '/home/oliver/Documents/MVPStudio/vpn/certs/ta.key' 1
client
remote '50.201.1.196'
auth-user-pass
cipher AES-128-CBC
dev tun
dev-type tun
proto tcp
port 443
nobind
auth-nocache
auth SHA256
script-security 2
persist-key
persist-tun
user nm-openvpn
group nm-openvpn
```

You must edit the first few paths to point to the certificates and keys provided to you by Continu. Once that's done you
can import this into the Network Manager via:

```
nmcli connection import type openvpn file <your .ovpn file>
```

If you then open any Network Manager gui you should see your new VPN connection.  Since authorization to Continu is a
mix of keys/certificates and username/password you must then also provide your username and password. If using a GUI
like `nm-connection-editor` or a tray applet you simply open the configuration and add your username and password.

## OSX

[Tunnelblick](https://tunnelblick.net/) seems to work well on OSX.

TODO: can someone who's set up the VPN via Tunnelblick provide some details??

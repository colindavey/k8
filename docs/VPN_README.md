# VPNs Plural

We have 2 VPNs. The first is an [OpenVpn](https://openvpn.net/) created by Continu for us. This gives us direct access
to all the machines in the cluster and we can ssh to individual nodes, etc. This is nice but it has a few downsides:

* Adding additional people to the VPN requires talking to Continu to set up certs and things.
* It provides a lot of access including the ability to send packets to individual nodes!

What we really want is the above VPN for MVP Studio admins but for regular users who just need to run `kubectl` we want
a VPN that limits their access to just the master server and we'd like to be able to add and remove new users easily and
without help from Continu. To this end we also have a simple [WireGuard](https://www.wireguard.com/) VPN.

# The WireGuard VPN

To access the [WireGuard](https://www.wireguard.com/) VPN users must first [install
WireGuard](https://www.wireguard.com/install/). Users then use WireGuard to generate a public and a private key. Keys
can be generated [as per the docs](https://www.wireguard.com/quickstart/#key-generation) though Windows and OSX clients
also have GUI ways to do this (TODO: users of these platforms please add details and screen shots).

## Windows Client
Once installed, you can add a new tunnel and create public/private keys by creating a new tunnel. 
![image](https://user-images.githubusercontent.com/311063/85208559-61b9a500-b2e6-11ea-9c2f-cea6515cb8ff.png)

Once keys are
generated users then generate a file like:

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


There's only 2 lines that need changing:

1. The `XXX` after `Address` has to be changed to be **unique** across all MVP Studio users. You can figure out what the
   next free address is by looking at [the server config file](../ansible/templates/wg0.conf).
2. Replace `<private key here>` with your actual private key.

We then need to install that user's public key on the server. To do that edit [the server config
file](../ansible/templates/wg0.conf) adding a new `[Peer]` section adding the user's public key and the unique IP
address they were assigned in step (1) above. Be sure to add a comment indicating what actual human the public key
belongs to. Finally, to apply the changed config use Ansible:


```
ansible-playbook -i inventory_wg.yml -l master --tags=wg_config --ask-vault-pass playbook.yml
```

You can find the password for the Ansible vault in the `continu-k8.txt` file in our secrets bucket
(`passwords.mvpstudio.org` bucket on GCE).

Note that `inventory_wg.yml` is the same as `inventory.yml` but the IP addresses are the addresses that show up in the
WireGuard tunnel. If you are on the OpenVpn tunnel use `inventory.yml` instead.

Finally, create a pull request to check in the modified `wg0.conf` file.

The master server now has IP address 100.64.0.1 on the WireGuard network.

Users should be able to test their WireGuard configuration as follows:

1. Bring up the interface: `wg-quick up /path/to/config.conf` where `/path/to/config.conf` is the client config file
   described above. As above, Windows and OSX offer GUI clients for this if desired.
2. Test the connection to the master via `ping 100.64.0.1` - you should get timely responses.
3. When down, break down the tunnel using a GUI or `wg-quick down /path/to/config.conf`

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

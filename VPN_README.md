# VPN

In order to access the cluster you must first connect to the VPN. To do that you will need Continu to set up a username,
password, and a variety of certificates and keys. They will general provide all this information to you on a thumb drive
or similar. Once you have the drive you'll need to configure some [OpenVpn](https://openvpn.net/) compatible client.

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

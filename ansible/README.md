# Overview

This directory contains our cluster inventory and an [Ansible](https://docs.ansible.com/) script to configure the nodes
in that inventory. Our initial setup was done manually so these scripts do not perform a complete, from-scratch, setup.
The manual steps that were performed are documented in [our hardware documentation](../docs/HARDWARE.md). However, if we
need to set up more minion nodes it's probably worthwhile to port them to Ansible so manual configuration is no longer
required.

## Running

To configure the nodes run:

```
ansible-playbook -i inventory.yml --ask-vault-pass playbook.yml
```

The password for the [Ansible vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) is in the Google
password bucket (`gs://passwords.mvpstudio.org`) in file `continu-k8.txt`. This allows us to decrypt the `vault.yml`
file which contains the passwords to log into each node, the password for the iSCSI targets, etc. This way you only
need to type a single password. If you need to update that file you can run `ansible-vault edit vault.yml`.

## On-Off Commands

While we primarily use Ansible to configure the cluster you can also use it to run commands on all (or a subset of all)
hosts. You can do that like so:

```
ansible -i inventory.yml all --ask-vault-pass -a 'hostname'
```

Note where it says `all`; you can run the command on a subset of the inventory by specifying a group, such as `minons`,
instead of `all`. You can also run Ansible modules other than `command` via the `-m` flag. For example, to get the
status of `/home/mpv/.bashrc` on all minions:

```
ansible -i inventory.yml minions --ask-vault-pass -m stat  -a 'path=/home/mvp/.bashrc'
```

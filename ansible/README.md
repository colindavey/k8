# Overview

This directory contains our cluster inventory and an [Ansible](https://docs.ansible.com/) script to configure the nodes
in that inventory. Our initial setup was done manually so these scripts do not perform a complete, from-scratch, setup.
The manual steps that were performed are documented in [our hardware documentation](../docs/HARDWARE.md). However, if we
need to set up more minion nodes it's probably worthwhile to port them to Ansible so manual configuration is no longer
required.

## Running

To configure the nodes run:

```
ansible -i inventory.yml -k playbook.yml
```

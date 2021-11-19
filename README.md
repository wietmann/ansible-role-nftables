Ansible Role: Nftables
======================

Ansible Role that installs and manages [nftables](https://wiki.nftables.org/wiki-nftables/index.php/Main_Page).

Ruleset is written in single file, directory `/etc/nftables` is created for manual include-files. On Debian systemd unit-file is patched to allow normal ruleset reload.

Requirements
------------

Python, Ansible >= 2.9

Role Variables
--------------

|Variable|Deafule value|Description|
|-|-|-|
|`nftables_validate_ruleset`|`True`|Wether to run `nft -c -f {{ nftables_ruleset_path }}`|
|`nftables_ruleset.include_files`|`[]`|List of include files path for generic level include statements|
|`nftables_ruleset.tables`|[]|List of tables|
|`nftables_ruleset.tables[*].name`|-|Table name, mandatory|
|`nftables_ruleset.tables[*].family`|`'ip'`|Table family: ip, arp, ip6, bridge, inet, netdev|
|`nftables_ruleset.tables[*].include`|`[]`|List of inlude file paths|
|`nftables_ruleset.tables[*].chains`|`[]`|List of chains in table|
|`nftables_ruleset.tables[*].chains[*].name`|-|Chain name, mandatory|
|`nftables_ruleset.tables[*].chains[*].type`|`'filter'`|Chain type: `'filter'`, `'route'`, `'nat'`|
|`nftables_ruleset.tables[*].chains[*].hook`|`'input'`|Chain hook|
|`nftables_ruleset.tables[*].chains[*].device`|-|Device associated with chain|
|`nftables_ruleset.tables[*].chains[*].policy`|`'accept'`|Chain policy action|
|`nftables_ruleset.tables[*].chains[*].priority`|`0`|Chain priority|
|`nftables_ruleset.tables[*].chains[*].rules`|`[]`|List of rule expressions|

Dependencies
------------

None.

Supported platforms
-------------------

* Debian
  * Buster (10)
  * Bullseye (11)
* RHEL
  * 7
  * 8

Example Playbook
----------------

```yaml
---
- hosts: all
  become: yes

  vars:
    nftables_ruleset:
      tables:
        - name: filter
          family: "ip"
          chains:
            - name: input
              type: "filter"
              hook: "input"
              policy: "drop"
              priority: 0
              rules:
                - "ct state established,related accept"
                - "iifname lo accept"
                - "icmp type echo-request accept"
                - "tcp dport {ssh} accept"
            - name: forward
              type: "filter"
              hook: "forward"
              policy: "drop"
              priority: 0
            - name: output
              type: "filter"
              hook: "output"
              policy: "accept"
              priority: 0

  roles:
    nftables
```

Testing
-------

This role uses [Molecule](https://molecule.readthedocs.io/en/stable/) for testing. Molecule is run against [Vagrant](https://www.vagrantup.com/docs)-managed VMs (see `default` scenario). By default, provider `libvirt` is used unless overriden by `MOLECULE_VAGRANT_PROVIDER` env variable.

Additional dependencies for controller host:
* Vagrant + preferred provider
* Python: `python-vagrant` library
* Python: `molecule-vagrant` [Molecule driver](https://github.com/ansible-community/molecule-vagrant)

Preferred way to test role for compatibility with different versions of Ansible is to use [Tox](https://tox.wiki/en/stable/). See `tox.ini` for the full list of supported Ansible versions.

Run tests for all environments:
```
tox
```

List available environments:
```
tox -l
```

Run tests only for specific environment (e.g. Ansible 2.9):
```
tox -e py3-ansible29
```

Run only specific Molecule subcommand (e.g. `molecule verify`):

```
# All test environments
tox -- molecule verify
# Specific test environments
tox -e py3-ansible29 -- molecule verify
```

License
-------

SPDX:MIT  
See full text in [LICENSE](LICENSE) file.

Author Information
------------------

This role was created in 2021 by Dmitry Danilov.

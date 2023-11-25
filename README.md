Role Name
=========

A customizable Firewall role for Debian 11 & 12 based on iptables alone. No scripts, no templates, no bullshit.

Requirements
------------

`iptables` and `iptables-persistent` should be installed or the role will handle the installation.

Role Variables
--------------

Most-used variables are listed here. See all available in `defaults/main.yml`.

```yaml
firewall_install: true
```

Whether or not to run the role.

```yaml
firewall_flush: true
```
Whether or not `iptables` and `ip6tables` should be flushed before applying any rules.

_To prevent accidental lockout, the `INPUT` policy is set to `ACCEPT` before flushing_.


```yaml
firewall_defaults: true
```

Whether or not the rules defined in `firewall_default_rules` should be applied.


```yaml
firewall_input_policy: "DROP"
firewall_forward_policy: "DROP"
firewall_output_policy: "ACCEPT"
```
Default firewall policies. **Be aware to not lock yourself out!**

```yaml
firewall_input_chain: "ansible_input"
firewall_output_chain: "ansible_output"
firewall_forward_chain: "ansible_forward"
```

Chain names to be used by this role. All rules/modification are always carried out in one of the three corresponding chains.


```yaml
firewall_dhcp_interface: "eth0"
```

When your server relies on DHCP to get an ip address, please specify the appropriate interface.
Leave empty if not needed.


```yaml
firewall_default_rules:
  - {
      ctstate: "ESTABLISHED,RELATED",
      jump: ACCEPT,
      comment: "Allow related (replies to already established connections)",
    }
  - {
      in_interface: lo,
      jump: ACCEPT,
      comment: "Allow all and everything on lo",
    }
  - {
      chain: "{{ firewall_output_chain }}",
      out_interface: lo,
      jump: ACCEPT,
      comment: "Allow all and everything on lo",
    }
  - { protocol: icmp, jump: ACCEPT, comment: "Allow icmp (ping)" }
  - { source_port: 53, protocol: tcp, jump: ACCEPT, comment: "Allow dns" }
  - { source_port: 53, protocol: udp, jump: ACCEPT, comment: "Allow dns" }
  - {
      protocol: udp,
      in_interface: "{{ firewall_dhcp_interface }}",
      source_port: 67:68,
      destination_port: 67:68,
      jump: ACCEPT,
      comment: "Allow dhcp",
    }
```

The default rules carried out if enabled with `firewall_defaults`.

```yaml
firewall_rules: []
# Allow ssh
# - { protocol: tcp, destination_port: 22, jump: ACCEPT }
# Allow http/https
# - { protocol: tcp, destination_port: 80, jump: ACCEPT }
# - { protocol: tcp, destination_port: 443, jump: ACCEPT }
# Allow Tor relay traffic
# - { protocol: tcp, destination_port: 9000, jump: ACCEPT }
# - { protocol: tcp, destination_port: 9001, jump: ACCEPT }
```

Here you can specify any custom rules. See `tasks/rule.yml` for all available options.
The following defaults are assumed if not specified:
  - `chain: firewall_input_chain`

Dependencies
------------

None.

Example Playbook
----------------

### Install locally
```yaml
# requirements.yml
- src: git+ssh://git@github.com/xtrcode/ansible-role-firewall.git
  scm: git
```

```bash
$ ansible-galaxy install -r requirements.yml -p ./roles
```

### Playbook
```yaml
- hosts: deploy
  roles:
    - { role: ansible-role-firewall }
  vars:
    firewall_install: true
    firewall_flush: true
    firewall_defaults: true
    firewall_rules:
      # Allow ssh
      - { protocol: tcp, destination_port: 22, jump: ACCEPT }
```

License
-------

MIT 

Author Information
------------------

Website: [blog.xtracode.ws](https://blog.xtracode.ws)

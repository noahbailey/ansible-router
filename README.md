# ansible-router
Simple Linux router implementation for Debian + Netfilter/IPtables

The purpose of this module is to quickly automate the process of provisioning and configuring a Linux router geared towards home/small business. 

| Currently Supported                                       | Planned Feature                     | Unplanned but neat                             |
| --------------------------------------------------------- | ----------------------------------- | ---------------------------------------------- |
| Network interfaces with IPv4 static or dynamic addressing | IPv6 support                        | Multi-factor authentication for OpenVPN server |
| VLAN trunking                                             | OpenVPN remote access server        | Automated IPsec config                         |
| IPtables firewall with default or custom rules            | Linux network stack tuning          | Dynamic routing - FRR or Quagga                |
| IPtables NAT with port-address translation                | DDNS with CloudFlare implementation | IDS/IPS - Snort of Suricata                    |
| IPTables for port-forwarding from WAN interface           |                                     | BIND9 dynamic DNS                              |
| isc-dhcp-server configuration                             |                                     |                                                |



## Usage

Add this role to the `roles` directory in your ansible project.

Then, include the role using a top level playbook:

```yaml
- name: Linux Router
  hosts: router
  become: true 
  roles: 
    - ansible-router
```



## Variables - Router

Minimum configuration: `eth0` connected to WAN and `eth1` connected to LAN. 

```yaml
router: 
  interfaces: 
    - name: eth0 
  	
    - name: eth1 
      cidr: 192.168.100.1/24 
```



When DHCP parameters are defined a DHCP server will be installed and configured: 

```yaml
router: 
  interfaces: 
    - name: eth0 

    - name: eth1 
      cidr: 192.168.100.1/24
      dhcp_start: 192.168.100.50
      dhcp_end: 192.168.100.250
```



The script can also use VLANs to run multiple networks over one link. In this case, WAN is on VLAN 666, and the inside network is on vlan 100. 

```yaml
router: 
  interfaces: 
    - name: eth0.666
    
    - name: eth0.100
      cidr: 192.168.100.1/24
      vlan: 100
      vlan_device: eth0
      dhcp_start: 192.168.100.50
      dhcp_end: 192.168.100.250
```



And of course, there can be multiple internal or external networks - so long as the Firewall rules are correctly configured. 



## Variables - Firewall 

If a simple stateful firewall is required, simply specify the inside and outside interfaces below the router block: 

```yaml
firewall: 
  outside: eth0
  inside:  eth1
```



If a more complex configuration is desired, rules can be specified in the `firewall.rules` configuration. 

```yaml
firewall: 
  rules: |
    *filter
    :INPUT DROP [0:0]
    :FORWARD DROP [0:0]
    :OUTPUT ACCEPT [0:0]
    -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT 
    -A INPUT -m comment --comment "Default deny rule" -j REJECT 
    -A FORWARD -m state --state RELATED,ESTABLISHED 
    -A FORWARD -i eth1 -o ens0 -m state --state NEW,RELATED,ESTABLISHED
```


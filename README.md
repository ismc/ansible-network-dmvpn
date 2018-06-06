# ansible-network-dmvpn

A role that deploys a DMVPN fabric over a hub and spoke network.

## Requirements

* netaddr

## Role Variables

* `dmvpn_node_type`: The node type being configured (hub or spoke)
* `dmvpn_loopback_number`: The loopback number added to the routers (Default: 0).
* `dmvpn_tunnel_number`: The tunnel number used for DMVPN (Default: 0).
* `dmvpn_tunnel_ip`: The IP address given to the tunnel on the devices being configured
* `dmvpn_loopback_ip`: The IP address given to the loopback interface on the device being configured.
* `dmvpn_hub_tunnel_ip`: The tunnel (inside) IP address of the Hub
* `dmvpn_hub_ip`: The outside IP address of the Hub
* `dmvpn_tunnel_interface`: The source interface to use for the tunnel
* `dmvpn_site_networks`: The list of site networks to advertise


## Dependencies


## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: spoke_routers
      gather_facts: no
      tasks:
        - name: Find the Hub Router
          set_fact:
            hub_router: "{{ groups.hub_routers[0] }}"

        - include_role:
            name: network-dmvpn
          vars:
            dmvpn_node_type: spoke
            dmvpn_tunnel_ip: "{{ dmvpn_tunnel_network | ipaddr(site_number|int + 1) }}"
            dmvpn_loopback_ip: "{{ dmvpn_loopback_network | ipaddr(site_number|int + 1) | ipaddr('address')}}/32"
            dmvpn_hub_tunnel_ip: "{{ dmvpn_tunnel_network | ipaddr(1) }}"
            dmvpn_hub_ip: "{{ hostvars[hub_router].ansible_host }}"
            dmvpn_tunnel_interface: "{{ interfaces.0.name }}"
            dmvpn_site_networks:
              - "{{ interfaces.1.ip | ipaddr('network') }}/{{ interfaces.1.ip | ipaddr('prefix') }}"

    - hosts: hub_routers
      gather_facts: no
      tasks:
        - include_role:
            name: network-dmvpn
          vars:
            dmvpn_node_type: hub
            dmvpn_tunnel_ip: "{{ dmvpn_tunnel_network | ipaddr(site_number|int + 1) }}"
            dmvpn_loopback_ip: "{{ dmvpn_loopback_network | ipaddr(site_number|int + 1) | ipaddr('address')}}/32"
            dmvpn_hub_tunnel_ip: "{{ dmvpn_tunnel_network | ipaddr(1) }}"
            dmvpn_tunnel_interface: "{{ interfaces.0.name }}"
            dmvpn_site_networks:
              - "{{ interfaces.1.ip | ipaddr('network') }}/{{ interfaces.1.ip | ipaddr('prefix') }}"



## License

GPL-3

## Author Information
* Steven Carter

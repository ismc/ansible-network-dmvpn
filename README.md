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

## Cloud Models

The cloud model is a cloud-agnostic data model that describes the particlar cloud deployment.  It creates the container network (e.g. VPC), the subnets, adds routes, security groups, etc.  It also contains instances, but this is mainly to deploy VNFs and service nodes.

### Cloud Model Example

    cloud_acls:
      host-acl:
        - { src_ip: 0.0.0.0/0, dst_ports: 22, proto: tcp }
        - { src_ip: 0.0.0.0/0, dst_ports: 443, proto: tcp }
        - { src_ip: 0.0.0.0/0, proto: icmp }
      rtr-acl:
        - { src_ip: 0.0.0.0/0, proto: all }
    vpc_list:
      - name: "{{ cloud_name }}"
        provider: "{{ cloud_provider }}"
        region: "{{ cloud_region }}"
        project: "{{ cloud_name }}"
        cidr: "{{ cloud_cidr }}"
        networks:
          - name: "outside.{{ cloud_name }}"
            cidr: "{{ cloud_cidr | ipsubnet(24, 0) }}"
            az: "{{ cloud_region }}a"
          - name: "inside.{{ cloud_name }}"
            cidr: "{{ cloud_cidr | ipsubnet(24, 1) }}"
            az: "{{ cloud_region }}a"
            routes:
              - cidr: "0.0.0.0/0"
                instance: "rtr1.{{ cloud_name }}"
                if_index: 2
        instances:
          - name: "rtr1.{{ cloud_name }}"
            size: medium
            image: csr-byol
            interfaces:
              - name: GigabitEthernet1
                subnet: "outside.{{ cloud_name }}"
                acl: rtr-acl
                public_ip: true
                private_ip: "{{ cloud_cidr | ipsubnet(24, 0) | ipaddr(-2) | ipaddr('address') }}"
              - name: GigabitEthernet2
                subnet: "inside.{{ cloud_name }}"
                acl: rtr-acl
                public_ip: true
                private_ip: "{{ cloud_cidr | ipsubnet(24, 1) | ipaddr(-2) | ipaddr('address') }}"
            tags:
              network_os: ios
              groups: network,routers,spoke_routers
            user_data: "{{ lookup('file', 'csr.user_data') }}"
          - name: "host1.{{ cloud_name }}"
            size: micro
            image: rhel7
            interfaces:
              - name: eth0
                subnet: "inside.{{ cloud_name }}"
                acl: host-acl
                private_ip: "{{ cloud_cidr | ipsubnet(24, 2) | ipaddr(10) | ipaddr('address') }}"
            tags:
              groups: hosts

## License

GPL-3

## Author Information
* Steven Carter

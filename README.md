Ansible Role: Customize VMware VCSA
=========

Customizes a newly deployed VCSA. The role attempts to configure the following:
- Creates a new datacenter and cluster and adds a set of hosts
- Configures NTP and SSH services to autostart
- Creates a DVS with two uplinks and creates vMotion and VM Network portgroups
- Migrates any VMs and the VMK0 interface to the DVS
- Enables DRS and HA
- Mounts NFS datastores

Requirements
------------

The role expects an environment that roughly conforms to these assumptions:
- There is an available VCSA to configure
- There is at least one ESXi host to add to the VCSA
- Each ESXi host has at least two 10Gb network adaptors
- All network plumbing is in place
- Hostvars files for each ESXi host denoting the vmnic names to use
- JMESPath, jq,

Role Variables
--------------

The following variables are defined in the defaults/main.yml file:

    # vCenter
    vcenter_hostname: "devvcsa.tme.nebulon.com"
    vcenter_datacenter: "SC0"
    vcenter_cluster: "AppFactory"
    # dvs
    # Name cannot contain a "-"
    vcenter_dvs_name: "dvs_appfactory"

This role defines the following variables in the vars/main.yml file:

    ntp_servers: [10.100.72.10,10.100.72.11]

    nfs_servers:
      - name: "ISO"
        nfs_mount_path: "/data/tme"
        nfs_server_ip: 10.100.24.8
      - name: backup
        nfs_mount_path: "/mnt/data/tme"
        nfs_server_ip: 10.100.24.202

    esxi_services:
      - ntpd
      - TSM
      - TSM-SSH

    suppressshellwarning: 1


    vcenter_hostname: "devvcsa.tme.nebulon.com"
    vcenter_datacenter: "SC0"
    vcenter_cluster: "AppFactory"
    vcenter_dvs_name: "dvs_appfactory"
    vcenter_dvs_version: "8.0.0"
    vcenter_dvs_mtu: "9000"
    vcenter_dvs_uplink_count: 2
    vcenter_dvs_uplink_prefix: "Uplink_"
    vcenter_dvs_discovery_protocol: lldp
    vcenter_dvs_disocvery_operation: both
    vcenter_dvs_folder: "/{{ vcenter_datacenter }}/network/{{ vcenter_dvs_name }}"
    vcenter_dvs_network_folder: "{{ vcenter_dvs_name }}"
    vcenter_dvs_vmotion_pg: "vmotion"
    vcenter_dvs_vmotion_vlan: 17
    vcenter_dvs_vmotion_vlan_trunk: false
    vcenter_dvs_vmnetwork_vlan: 0
    vcenter_dvs_vmnetwork_vlan_trunk: false
    vcenter_dvs_portgroup_vmnetwork: "vmnetwork"
    vcenter_dvs_portgroup_lb_policy: loadbalance_loadbased

    # Vault credentials
    vcenter_username: "{{ vault_vcenter_username }}"
    vcenter_password: "{{ vault_vcenter_password }}"
    esxi_username: "{{ vault_esxi_username }}"
    esxi_password: "{{ vault_esxi_password }}"

Additionally, there are four other variables for the vMotion network and vmnic device names of the 10GB interfaces used when configuring the DVS. The role expects these variables to be available in hostvars and I suggest placing them in a host_vars file per host. These could be placed in inventory as well.

    vmotionip: "10.100.28.46"
    vmotionnetmask: "255.255.255.0"
    d0_vmnic: "vmnic0"
    d1_vmnic: "vmnic1"

Dependencies
------------

Ansible community.vmware collection [v3.2.0+](https://galaxy.ansible.com/community/vmware)

Example Playbook
----------------

    # ===========================================================================
    # Customize the VCSA installation
    # ===========================================================================
    - name: Customize VCSA
      hosts: localhost
      connection: local
      module_defaults:
        group/vmware:
          hostname: '{{ vcenter_hostname }}'
          username: '{{ vcenter_username }}'
          password: '{{ vcenter_password }}'
          validate_certs: false
        vmware_host:
          datacenter_name: '{{ vcenter_datacenter }}'
          cluster_name: '{{ vcenter_cluster }}'
          esxi_username: '{{ esxi_username }}'
          esxi_password: '{{ esxi_password }}'
        vmware_host_service_manager:
          cluster_name: '{{ vcenter_cluster }}'
        vmware_host_config_manager:
          cluster_name: '{{ vcenter_cluster }}'
        vmware_datastore_info:
          datacenter_name: '{{ vcenter_datacenter }}'

      vars_files:
        # Ansible vault with all required passwords
        - "../../credentials.yml"

      roles:
        - jedimt.vmware_vcsa_customization

License
-------

MIT

Author Information
------------------

Aaron Patten
aaronpatten@gmail.com

# ansible-vmware-new-vm

Role to create new virtual machine (Linux based) using VMware vCenter API.
This role was created with AWX (Ansible Tower) in mind and to work with multiple VMware vCenters.

>e.g.: You have implemented a geo redundancy architecture and installed two VMware vCenter one for `NorthDatacenter` other for `SouthDatacenter`. Just need to pass `vm_vcenter_host` in your inventory.

Will use a single source of truth, versioned inventory file located on your favorite git server.

The approach taken was to use that inventory file for VM deployment and dynamic inventory to config afterwards.

That means that everytime your job (in case of AWX) runs, will check your source of truth, and will implement the state that you have there described (running state, memory, etc etc).

Features that you can edit will be described below at `Role Variables`

**Known limitations:**
* Network is not dynamic as HDD list
  * Need to add/remove element `networks` dict at main tasks
* Working with defaults, disks are dynamic up to 4 (1 OS + 3 DATA):
  * Need to add/remove element `disk_dict` dict at main defaults
  * Need to add/remove key/value `vm_extradisk_size_gb` at main defaults

Maybe in future that could be changed - if time allows it - to a beautiful Jinja2 template

**Note:** Features like CPU/RAM hotplug, and correct usage with Ansible, will always depend on your VMware vCenter/Hardware. 

## Installation

Clone repo and run from your own playbook.

## Requirements

https://docs.ansible.com/ansible/latest/modules/vmware_guest_module.html:
* python >= 2.6
* PyVmomi

To fully integrated with this role, out of box, can use AWX and need to:
* Setup vCenter credentials and map to your job

To be used without AWX:
* Setup VMWARE_* vars on your environment session

VMware vCenter:
* You need to have credentials to proceed with auth
* You need to have a Template created to proceed with clone

## Role Variables

### Vars - connection to vCenterList:

Like mentioned before, to be used with AWX, change accordingly to your needs:

| Variable        | Description                                      | Defaults                           |
|-----------------|--------------------------------------------------|------------------------------------|
| vm_vcenter_host | VWware IP/FQDN to be added at inventory, if need | *Optional* defaults to VMWARE_HOST |
| VMWARE_HOST     | VWware IP/FQDN                                   | setup shell/session env            |
| VMWARE_USER     | VWware User to connect to vCenter                | setup shell/session env            |
| VMWARE_PASSWORD | VWware User password to connect to vCenter       | setup shell/session env            |

### Vars - vCenter common setup:

Specific to vCenter, you gonna need to setup where you want to deploy VM and map to cluster/folders/etc etc:

| Variable              | Description                             | Defaults |
|-----------------------|-----------------------------------------|----------|
| vm_vcenter_datacenter | VWware vCenter Datacenter to deploy VM  | *None*   |
| vm_vcenter_cluster    | VMware vCenter Cluster to deploy VM     | *None*   |
| vm_vcenter_folder     | VMware vCenter folder to deploy VM      | *None*   |
| vm_vcenter_template   | VMware vCenter image to clone           | *None*   |


### Vars - VM specs:

Specific to VM to be deployed:

| Variable                 | Description                             | Defaults           |
|--------------------------|-----------------------------------------|--------------------|
| vm_hostname              | Hostname to be configured at VM         | inventory_hostname |
| vm_ram                   | Amount of VM RAM                        | 2048               |
| vm_num_cpus              | Amount of VM CPU                        | 2                  |
| vm_dns_server            | DNS server to be configured             | *None*             |
| vm_dns_suffix            | DNS suffix to be configured             | *None*             |
| vm_eth0_network          | ETH0 PG                                 | *None*             |
| vm_eth0_ip               | ETH0 IP                                 | *None*             |
| vm_eth0_mask             | ETH0 MASK                               | *None*             |
| vm_eth0_gw               | ETH0 GW                                 | *None*             |
| vm_vcenter_vds           | DSwitch                                 | *None*             |
| vm_eth0_device           | Device emulation                        | *None*             |
| vm_vcenter_net_connected | Network connection enabled after deploy | False              |
| vm_disk_datastore        | Where to store VM                       | *None*             |
| vm_extradisk_size_gb2    | Number in GB for extra disk             | *None*             |

:wave: :warning: Repeat every `vm_eth0_ip' times the amount of network interfaces (by default you need to setup eth0 and eth1 vars)

:wave: :warning: By default it will suport 4 disks (1 for OS plus 3 for DATA) and you will need to fill accordingly in your inventory `vm_extradisk_size_gbX`, but only declare the ones you need to. It will create the ones that you declare only (up to a total of 4, by default)

**Big Note:** if you need to, I would suggest to edit the code to have defaults or even better... just create groups at inventory and setup a var for a specific group. e.g.: Datacenter

## Example Playbook

Create a inventory file and load when running with playbook.

Inventory file example:

```ini
    [mynewvm]
    mynewvmName

    [mynewvm:vars]
    vm_vcenter_net_connected=True
    vm_vcenter_template=templateCentos7
    vm_vcenter_cluster=VMwareCluster
    vm_vcenter_vds=vDS_vcenter
    vm_eth0_network=portgroup_network01
    vm_eth0_ip=10.0.0.100
    vm_eth0_mask=255.255.255.0
    vm_eth0_gw=10.0.0.1
    vm_eth1_network=portgroup_network02
    vm_eth1_ip=10.0.1.100
    vm_eth1_mask=255.255.255.0
    vm_eth1_gw=10.0.1.1
    vm_extradisk_size_gb2=100
```

## License

GNU GPLv3

## Author Information

* Joao Santos (morgado.santos@gmail.com)

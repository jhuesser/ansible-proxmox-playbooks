# Proxmox Autoprovisioning from Netbox with ansible
Playbooks for Proxmox automation - Work in progress

# ToDo list

- [ ] Integrate into [Semaphore](https://github.com/semaphoreui/semaphore)
- [ ] Integrate Semaphore into [Netbox](https://github.com/netbox-community/netbox) Webhook
- [ ] Integrate package installation for the provisioned VM
- [ ] Integrate usage of vault instead of `.env` file
- [ ] Integrate auto decommissioning on delete ([dangerzone!](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExa201eGNwZzd1bjYwMWQ0dzY2MmNrcW5vNDc4bzBtdDZxdzE1bmJpbCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/H8iL56bXGjVE4/giphy.gif))


# Prerequisites 
## [Netbox](https://github.com/netbox-community/netbox)

Netbox must be fully setup and configured for this to work. We use different features of netbox, here is what you need to have:

### Devices

You need to create at least one proxmox host as device. The following fields must be present:

- Description (This must be the node name in proxmox)
- Primary IP assigned

You also need to create a platform for each VM OS you're going to use, aswell as each LXC template. LXC platforms need to start with `lxc-`

### IP-address

The primary IP-address assigned to the proxmox device in netbox, needs the following fields set:

- dns_name: This is the FQDN which is used for the proxmox API communication.


### Virtualization

- Cluster type: Just create "Proxmox"
- Clusters: Create your pve cluster

Assign the proxmox hosts you created as device before to the newley created cluster.

### Tags

You need to create the following tags:

| tag name | applies to | description |
| -------- | ---------- | ----------- |
| <pve_datastore_name> | Virtual Disks | This tag is applied to virtual disks to determine on which datastore they get provisioned. |
| no-auto-provisioning | Virtual machines | In the future this tag is used to skip the creation of certein VMs |
| pve-provisioned | Virtual machines | This tag will be applied once the VM is created, to prevent duplications |


### Provisioning

Create a config template for each LXC template you're using. The property name is `proxmox_template` and the value is the unique identifier of the template, eg:

```json
{
    "proxmox_template": "local:vztmpl/ubuntu-24.04-standard_24.04-2_amd64.tar.zst"
}
```

You need to map this config template to the corresponding plattform you've created before.

### Admin
Create an API token with write access and save it.

## [Proxmox](https://proxmox.com/en/proxmox-virtual-environment/overview)

### VMs

you need to create a VM template, with [cloud-init](https://cloud-init.io) enabled, in proxmox. Make sure, that the template name equals to the platform slug in netbox.

### LXC

Download all LXC templates you want to use. Map them to config templates in netbox as mentioned above

### Admin stuff
Create a API token for root@pam and safe it.
The FQDN of proxmox needs a trusted TLS certificate.




# How to run this thing?

Currently only manual run is possible, in the future it will trigger a semaphore webhook, but I first need to set this up and document it.

So meanwhile, you can do the following:

1. Create a virtual machine in netbox. Required fields:
- name
- cluster
- device (for now)
- platform
- primary IPv4
- config template (only if LXC)
- vCPUs
- Memory
- Disk
2. Add a virtual disk. required fields:
- virtual machine
- Size
- Tag (use the datastore tag)
3. Copy the `.env.dist` file to `.env` and fill in your netbox and proxmox API token, aswell as your netbox FQDN.
4. Set the `NETBOX_VMLXC_ID`env variable to the VM ID in netbox. You can find it in the URL.
5. Run `ansible-playbook test-vm-staging-with-lxc.yml` 


# See also
If you want your DNS records in netbox auto synced to your provider, have a look at my [Netbox-OctoDNS-syncer](https://github.com/jhuesser/octodns-webhook-listener/tree/main)
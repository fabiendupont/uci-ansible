# IMS > Universal Conversion Image > Ansible stuff

This repo provides a role to deploy and configure a conversion host based on
Universal Conversion Image (UCI).

__Note__: The deployment currently supports only oVirt/RHV.

## Installation

The installation simply consists in cloning this repository:

```
# git clone https://github.com/fdupont-redhat/uci-ansible.git
```


__Note__: To run the playbook, you will need to manually  install the
`manageiq.v2v-conversion-host` role, as it requires a specific version. Here
are the steps:

```
# cd /opt
# git clone https://github.com/ManageIQ/manageiq-v2v-conversion_host.git
# cd /usr/share/ansible/roles
# ln -s /opt/manageiq-v2v-conversion_host/ansible/manageiq.v2v-conversion-host
```


## Playbooks

There's currently only one playbook: (deploy.yml). It creates the conversion
host VM and then configures it. You only have to create the variable file(s).

__Note__: For security reasons, we advise to store sensitive data in a vault.

```
# vim ~/vars.yml
# ansible-vault create ~/vault.yml
# ansible-playbook -i localhost, -e @~/vars.yml -e @~/vault.yml --ask-vault-pass deploy.yml
```

Once the conversion host VM is configured, it can run a conversion.

## Role Variables

### Variables for the conversion host VM

| Variable            | Default value | Description                                                                    |
| ------------------- | ------------- | ------------------------------------------------------------------------------ |
| uci_disk_image_url  |               | URL of conversion host appliance QCOW2 image                                   |
| uci_disk_image_size |               | Conversion host appliance image size                                           |
| uci_vm_name         |               | Name of the conversion host VM                                                 |
| uci_cpu_sockets     | 2             | Number of CPU sockets for conversion host VM                                   |
| uci_cpu_cores       | 2             | Number of CPU cores per socket for conversion host VM                          |
| uci_memory          | 8GiB          | Memory allocation for conversion host VM                                       |
| uci_subnet_cidr     |               | Subnet CIDR notation of the network to which conversion host VM is connected   |
| uci_ssh_public_key  |               | SSH public key that is added to cloud-user's authorized keys during deployment |


### Variables for the provider

The provider is the platform where the conversion host is deployed. Currently,
we support only oVirt. The variables are all prefix by the provider type.

| Variable | Default value | Description |
| ---------------- | --------------------- | ---------------------------------- |
| provider_type    |           | Type of the provider. Valid values: `ovirt`, `openstack`. |


#### Variables for oVirt

| Variable             | Default value  | Description                                                                    |
| -------------------- | -------------- | ------------------------------------------------------------------------------ |
| ovirt_hostname       |                | Hostname or IP address of oVirt Manager                                        |
| ovirt_username       | admin@internal | Username to connect to oVirt Manager                                           |
| ovirt_password       |                | Password to connect to oVirt Manager                                           |
| ovirt_ca_file        |                | Path of CA certificates bundle file used to verify oVirt Manager's certificate. On oVirt Manager, it is `/etc/pki/ovirt-engine/apache-ca.pem` |
| ovirt_cluster        |                | oVirt cluster where to create the conversion host VM                           |
| ovirt_storage_domain |                | oVirt storage domain where to upload the conversion host disk                  |
| ovirt_vnic_profile   | ovirtmgmt      | oVirt vNIC profile to which attach the conversion host NIC                     |


#### Variables for OpenStack

| Variable                      | Default value  | Description                                                       |
| ----------------------------- | -------------- | ----------------------------------------------------------------- |
| openstack_hostname            |                | Hostname or IP address of OpenStack public API endpoint           |
| openstack_username            |                | USername to connect to OpenStack public API                       |
| openstack_password            |                | Password to connect to OpenStack public API                       |
| openstack_user_domain_name    |                | Name of the keystone domain for users                             |
| openstack_project_domain_name |                | Name of the keystone domain for projects                          |
| openstack_project_name        |                | Name of the project in which the conversion host will be deployed |
| openstack_image_name          |                | Name of the image under which the UCI disk image will be uploaded |
| openstack_flavor              |                | Flavor to use to create the conversion host                       |
| openstack_keypair             |                | SSH key pair to use to create the conversion host                 |
| openstack_network             |                | Network to which attach the conversion host NIC                   |
| openstack_security_groups     |                | Security groups to apply to the conversion host                   |

We consider that the OpenStack environment is ready to upload the image and
create the conversion host. This can be automated by an Ansible role or
playbook, but this is beyond our scope.


### Variables for manageiq.v2v-conversion-host role

| Variable                       | Default value | Description                                                                                                                   |
| -----------------------------  | ------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| v2v_transport_method           |               | Transport method to configure on the conversion host. Valid values: `vddk`, 'ssh`.                                            |
| v2v_vddk_package_url           |               | URL to the VDDK library package.                                                                                              |
| v2v_vddk_override              | false         | Normally the install role is not run if the plugin is already installed. To force the deployment set this variable to `true`. |
| v2v_ssh_private_key            |               | The private key to use to connect to the VMware host.                                                                         |
| v2v_ca_bundle                  |               | A bundle of CA certificates to allow connection to the provider where the conversion host belongs. See below for value.       |


### Examples variable files

#### Example variable file for oVirt

```yaml
---
# The type of provider where we deploy the conversion host
provider_type: ovirt

# The provider connection details
ovirt_hostname: ovirt.example.com
ovirt_username: admin@internal
ovirt_password: secret
ovirt_ca_file: /root/rhv_ca.pem

# The conversion host placement information
ovirt_cluster: uci_cluster
ovirt_storage_domain: uci_data
ovirt_vnic_profile: uci_network

# The conversion host VM details
uci_disk_image_url: http://content.example.com/uci/v2v-conversion-host-appliance-latest.qcow2
uci_disk_image_size: 10GiB
uci_vm_name: my_uci
uci_subnet_cidr: "192.168.0.0/24"
uci_ssh_public_key: "ssh-rsa AAAAB...YZZZZ UCI Public Key"

# The conversion configuration (see. https://github.com/ManageIQ/manageiq-v2v-conversion_host/blob/master/docs/Ansible.md)
v2v_transport_method: vddk
v2v_vddk_package_url: http://content.example.com/vddk/VMware-vix-disklib-stable.tar.gz
v2v_ca_bundle: |
  -----BEGIN CERTIFICATE-----
  MIID4TCCAsmgAwIBAgICEAAwDQYJKoZIhvcNAQELBQAwUjELMAkGA1UEBhMCVVMx
  [...]
  SGCRHHJld1kAiQUJnHTeArOIhflg4IW+bxG/skxt6r/oatuv6g==
  -----END CERTIFICATE-----
```

#### Example variable file for oVirt

```yaml
---
# The type of provider where we deploy the conversion host
provider_type: openstack

# The provider connection details
openstack_hostname: openstack.example.com
openstack_username: admin
openstack_password: secret
openstack_user_domain_name: Default
openstack_project_domain_name: Default
openstack_project_name: migration

# The conversion host placement information
openstack_image_name: ims-uci
openstack_flavor: ims.conversion-host
openstack_keypair: migration
openstack_network: external_network
openstack_security_groups:
  - ims.conversion-host

# The conversion host VM details
uci_disk_image_url: http://content.example.com/uci/v2v-conversion-host-appliance-latest.qcow2
uci_disk_image_size: 10GiB
uci_vm_name: my_uci
uci_subnet_cidr: "192.168.0.0/24"
uci_ssh_public_key: "ssh-rsa AAAAB...YZZZZ UCI Public Key"

# The conversion configuration (see. https://github.com/ManageIQ/manageiq-v2v-conversion_host/blob/master/docs/Ansible.md)
v2v_transport_method: vddk
v2v_vddk_package_url: http://content.example.com/vddk/VMware-vix-disklib-stable.tar.gz
v2v_ca_bundle: |
  -----BEGIN CERTIFICATE-----
  MIID4TCCAsmgAwIBAgICEAAwDQYJKoZIhvcNAQELBQAwUjELMAkGA1UEBhMCVVMx
  [...]
  SGCRHHJld1kAiQUJnHTeArOIhflg4IW+bxG/skxt6r/oatuv6g==
  -----END CERTIFICATE-----
```

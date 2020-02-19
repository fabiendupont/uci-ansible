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

| Variable            | Default value | Description                                                   |
| ------------------- | ------------- | ------------------------------------------------------------- |
| uci_disk_image_url  |               | URL of conversion host appliance QCOW2 image                  |
| uci_disk_image_size |               | Conversion host appliance image size                          |
| uci_vm_name         |               | Name of the conversion host VM                                |
| uci_cpu_sockets     | 2             | Number of CPU sockets for conversion host VM                  |
| uci_cpu_cores       | 2             | Number of CPU cores per socket for conversion host VM         |
| uci_memory          | 8GiB          | Memory allocation for conversion host VM                      |
| uci_subnet          |               | Subnet CIDR notation to which conversion host VM is connected |
| uci_ssh_public_key  |               | SSH public key that is added to cloud-user's authorized keys  |


### Variables for the provider

The provider is the platform where the conversion host is deployed. Currently,
we support only oVirt. The variables are all prefix by the provider type.

| Variable | Default value | Description |
| ---------------- | --------------------- | ---------------------------------- |
| provider_type    |           | Type of the provider. Valid values: `ovirt`. |


#### Variables for oVirt

| Variable             | Default value  | Description                                                              |
| -------------------- | -------------- | ------------------------------------------------------------------------ |
| ovirt_hostname       |                | Hostname or IP address of oVirt Manager                                  |
| ovirt_username       | admin@internal | Username to connect to oVirt Manager                                     |
| ovirt_password       |                | Password to connect to oVirt Manager                                     |
| ovirt_ca_file        |                | CA certificate(s) bundle file used to verify oVirt Manager's certificate |
| ovirt_cluster        |                | oVirt cluster where to create the conversion host VM                     |
| ovirt_storage_domain |                | oVirt storage domain where to upload the conversion host disk            |
| ovirt_vnic_profile   | ovirtmgmt      | oVirt vNIC profile to which attach the conversion host NIC               |


### Variables for manageiq.v2v-conversion-host role

| Variable                       | Default value | Description                                                                                                                   |
| -----------------------------  | ------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| v2v_transport_method           |               | Transport method to configure on the conversion host. Valid values: `vddk`, 'ssh`.                                            |
| v2v_vddk_package_url           |               | URL to the VDDK library package.                                                                                              |
| v2v_vddk_override              | false         | Normally the install role is not run if the plugin is already installed. To force the deployment set this variable to `true`. |
| v2v_ssh_private_key            |               | The private key to use to connect to the VMware host.                                                                         |
| v2v_ca_bundle                  |               | A bundle of CA certificates to allow connection to the provider where the conversion host belongs. See below for value.       |


### Example variable file

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
uci_subnet: "192.168.0.0/24"
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


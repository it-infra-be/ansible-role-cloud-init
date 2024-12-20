# Ansible Role: Cloud Init

This role will install and configure all components needed to use the NoCloud Cloud Init datasource:
https://cloudinit.readthedocs.io/en/latest/reference/datasources/nocloud.html

The cloud-init configuration files will be provided over HTTP (NGINX).

## Requirements

This role has to be executed as user 'root'.

## Variables

| Variable                   | Type   | Required | Default    | Comment                             |
|----------------------------|--------|----------|------------|-------------------------------------|
| cloud_init_ipv4            | string | No       | *          | IPv4 interface to listen on (NGINX) |
| cloud_init_ipv6            | string | No       | [::]       | IPv6 interface to listen on (NGINX) |
| cloud_init_port            | int    | No       | 80         | Port to listen on (NGINX)           | 
| cloud_init_firewalld_zones | list() | No       | ['public'] | Firewalld zones to allow cloudinit  |
| cloud_init_user_data       | dict() | No       | N/A        | Cloud-config user-data (multiple)   |
| cloud_init_vendor_data     | dict() | No       | N/A        | Cloud-config vendor-data (global)   |

### Cloud Config

Both vendor-data and user-data are provided in the 'cloud config' YAML format:
https://cloudinit.readthedocs.io/en/latest/explanation/format.html#cloud-config-data

The user-data contains one or more keys, each defining a separate user-data configuration.
Each key defines the URI path to use to reach the specific user-data configuration file.

The vendor-data is defined globally and added to all user-data directories.

```yaml
cloud_init_vendor_data:
  # vendor-data configuration
  ssh_pwauth: false

cloud_init_user_data:
  default:
    # user-data configuration
    users:
      - name: ansible
        gecos: Ansible User
        sudo: ALL=(ALL) NOPASSWD:ALL
        lock_passwd: true
        ssh_authorized_keys:
          - ssh-ed25519 AAAAC5NzaC1lZDI1NTE5AAAAIMFrSW12itEoIMaREyhMXCV70I+KEo3IdSx4TXIgGTRF
```

**Note:** An empty meta-data file is also provided in each user-data directory in order to make cloud-init work.

## Examples

### Vendor- and user-data

The above cloud-init configuration can be reached via 'http://IP_ADDRESS/default' and will provide three files.

The globally defined vendor-data file:

```yaml
#cloud-config
ssh_pwauth: false
```

The specific user-data file:

```yaml
#cloud-config
users:
  - name: ansible
    gecos: Ansible User
    sudo: ALL=(ALL) NOPASSWD:ALL
    lock_passwd: true
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC5NzaC1lZDI1NTE5AAAAIMFrSW12itEoIMaREyhMXCV70I+KEo3IdSx4TXIgGTRF
```

And an empty meta-data file.

### Libvirt

Following Libvirt domain configuration can be used to use the Cloud-init configuration during boot:

```xml
  <os>
    <type arch='x86_64' machine='pc-q35-rhel9.4.0'>hvm</type>
    <smbios mode='sysinfo'/>
  </os>
  <sysinfo type='smbios'>
    <system>
      <entry name='serial'>ds=nocloud-net;s=http://IP_ADDRESS/default/</entry>
    </system>
  </sysinfo>
```

> [!WARNING]
> Do not use RHEL 9.4 images as these contain a cloud-init version which do not work with the 'nocloud' seedfrom URL!
> https://bugzilla.redhat.com/show_bug.cgi?id=2279305

# Ansible & Vagrant

> [!NOTE]
> This repo is a fork of
> [jborean93/ansible-windows](https://github.com/jborean93/ansible-windows) with
> some changes (see below).

The Vagrantfile is primarily inventory driven and is written to work with
libvirt provider. You can use it to spin up a single Windows or Linux host
or a standalone domain that contains a mixture of Windows and Linux hosts

## Requires

* Vagrant
* Libvirt and the vagrant-libvirt plugin
* pypsrp

## The Inventory

The inventory controls how hosts are created and provisioned. Along with
normal Ansible host variables, you can define libvirt provider options here.
These are options that would normally be hardcoded in the Vagrantfile.

For example

```yaml
provider_options:
  cpus: 4
  memory_mb: 4096
  cpu_mode: host-passthrough
  nic_model: virtio
  graphics:
    type: spice
  video_type: qxl
  memory_backing: "off"
  sync_folders: []
```

These values are read by the Vagrantfile and applied accordingly.

## The Vagrantfile

If you want to support additional provider options, you can extend
`PROVIDER_OPTIONS_ALLOWED`.

For example, to add support for CPU topology

```ruby
PROVIDER_OPTIONS_ALLOWED = %w[
  ...
  ...
  cpu_topology
].freeze
```

Then update the `apply_provider_options` function and append something like

```ruby
if opts['cpu_topology'].is_a?(Hash)
  topo = opts['cpu_topology']
  lv.cputopology(
    sockets: Integer(topo['sockets']),
    cores: Integer(topo['cores']),
    threads: Integer(topo['threads'])
  )
end
```

And define it in the inventory

```yaml
provider_options:
  ...
  cpu_mode: host-model
  cpu_topology:
    sockets: 2
    cores: 2
    threads: 1
  ...
```

You can add validation, defaults, or better error handling if you want.
The idea is to keep things flexible and let the inventory drive what gets
exposed.

> [!NOTE]
> Vagrant libvirt documentation reference can be found
> [here](https://vagrant-libvirt.github.io/vagrant-libvirt/configuration.html)

## The Roles

Each role has its own README, a short summary of what is included:

* adcs-enrollment

  * Installs and configures Active Directory Certificate Services and sets up
    a GPO so all domain machines automatically enroll for certificates.

* adcs-winrm

  * Updates the existing WinRM HTTPS listener to use the auto-enrolled
    certificate issued by AD CS.

* ansible-setup

  * Creates a Python 3 virtual environment in the user's home directory with
    Ansible and pywinrm installed. It also pulls the latest devel checkout and
    loads it via the shell.

* domain-join

  * Joins a host to an existing domain based on inventory variables.

* domain-setup

  * Creates a new Windows domain. This is a minimal setup with no hardening
    or security best practices applied.

* kerberos

  * Installs and configures Kerberos packages, DNS, and realm settings so the
    host can authenticate against the domain.

* python

  * Builds and installs Python using `make altinstall` under
    `/usr/local/bin/python*`.

* user-setup

  * Creates an admin user and installs an SSH key based on inventory variables.

* guest-tools

  * Installs spice-guest, winfsp, virtiofs tools, and enables OpenSSH server capability.

## Examples

To bring up a standalone domain controller (dc01) and join another Windows
host (win2022)

```sh
vagrant up dc01 win2022 --provision
```

This creates a domain environment with

* A domain user defined in `inventory.yml` that is a member of Domain Admins
* Active Directory Certificate Services with automatic certificate enrollment
* A local `ca_chain.pem` file containing the root CA certificate
* Child hosts joined to the domain
* WinRM HTTPS listeners using certificates issued by AD CS

> [!NOTE]
> Although Vagrant boxes are created in parallel, if dc01 is included,
> Ansible provisioning ensures it runs first. A marker file is created at
> `.vagrant/markers/dc_ready`. When dc01 is destroyed, this marker is cleaned up.

> [!IMPORTANT]
> When destroying the controller instance, always use `vagrant destroy`.
> The marker cleanup logic runs as an after trigger in the Vagrantfile.

To bring up standalone Windows boxes without provisioning

```sh
vagrant up win2022 --no-provision
```

You can also target inventory groups using `VAGRANT_GROUP`

```sh
VAGRANT_GROUP=linux \
vagrant up --no-provision
```

`VAGRANT_GROUP` supports sub-groups and individual hosts as well

```sh
VAGRANT_GROUP=controller,domain_children,ubuntu24 \
vagrant up --provision
```

In addition to Vagrant options and `VAGRANT_GROUP`, you can pass extra flags
using `ANSIBLE_EXTRA_ARGS`. These arguments are passed straight through to
ansible-playbook, so, ideally any valid `ansible-playbook` option should work.

For example

```sh
ANSIBLE_EXTRA_ARGS="-vv --forks=2" \
VAGRANT_GROUP=windows \
vagrant up --provision
```

> [!NOTE]
> For linux boxes, if you run provisioning directly using `ansible-playbook`
> instead of `vagrant provision`, you may have to specify the private key
> explicitly. It is usually located at `.vagrant/machines/[box]/libvirt/private_key`.

> [!WARNING]
> Windows boxes take much longer to download and provision than Linux boxes.
> Some patience helps here.

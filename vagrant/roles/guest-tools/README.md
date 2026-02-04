# guest-tools role

## Overview

This role installs common **Windows guest tools** for VMs running on KVM

---

## Variables

All variables are optional

### General

| Variable | Default | Description |
|---|---:|---|
| `temp_dir` | `C:\Temp` | Temporary directory used for downloads |

### VirtIO guest tools

| Variable | Default | Description |
|---|---:|---|
| `virtio_version` | `0.1.266-1` | VirtIO archive version |
| `virtio_base_url` | `https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio` | Base URL for VirtIO |
| `virtio_gt_filename` | `virtio-win-gt-x64.msi` | VirtIO guest tools MSI filename |

> internal: `virtio_gt_url`, `virtio_gt_local`, `virtio_log_path`, `virtio_task_name`

### WinFSP

| Variable | Default | Description |
|---|---:|---|
| `winfsp_version` | `2.0.23075` | WinFSP version |
| `winfsp_base_url` | `https://github.com/winfsp/winfsp/releases/download/v2.0` | Base URL for WinFSP |

> internal: `winfsp_url`

### SPICE guest tools

| Variable | Default | Description |
|---|---:|---|
| `spice_url` | `https://www.spice-space.org/download/windows/spice-guest-tools/spice-guest-tools-latest.exe` | SPICE Guest Tools installer URL |

---

## Tags

* `win_tools` - install SPICE, WinFSP, and VirtIO guest tools
* `win_ssh` - install and configure OpenSSH Server

---

## Examples

Run the role

```yaml
- name: Install windows guest tools
  hosts: windows
  gather_facts: false
  roles:
    - guest-tools
```

Install guest tools

```bash
ansible-playbook playbooks/guest-tools/main.yml \
  --inventory vagrant/inventory.yml \
  --limit dc01 \
  --tags win_tools \
  -vv
```

Install OpenSSH

```bash
ansible-playbook playbooks/guest-tools/main.yml \
  --inventory vagrant/inventory.yml \
  --limit dc01 \
  --tags win_ssh \
  -vv
```

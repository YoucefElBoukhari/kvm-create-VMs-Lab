# KVM VM Provisioning with Ansible

An Ansible playbook and role to automate the creation of KVM (Kernel-based Virtual Machine) virtual machines on a libvirt host. Instead of manually running `virt-install` or clicking through `virt-manager`, this lab lets you spin up fully configured VMs from a cloud image with a single command.

## Features

- Automated download and preparation of a base cloud image (qcow2)
- VM definition generated from a Jinja2 libvirt XML template
- Configurable CPU, memory, disk, and network settings
- Idempotent runs — re-running the playbook won't recreate an existing VM
- Cloud-init friendly (SSH key injection, hostname, etc.)
- Clean, reusable Ansible role structure

## Project Structure

```
kvmlab/
├── kvm_provision.yaml              # Main playbook (entry point)
└── roles/
    └── kvm_provision/
        ├── defaults/
        │   └── main.yml            # Default variables (override these)
        ├── meta/
        │   └── main.yml            # Role metadata
        ├── tasks/
        │   ├── main.yml            # Main task list
        │   └── vm_install.yml      # VM creation logic
        ├── templates/
        │   └── vm-template.xml.j2  # libvirt domain XML template
        ├── tests/
        │   ├── inventory           # Test inventory
        │   └── test.yml            # Test playbook
        └── README.md               # Role-level documentation
```

## Requirements

On the **KVM host** (the target machine that will run the VMs):

- A Linux host with hardware virtualization enabled (Intel VT-x / AMD-V)
- `libvirt`, `qemu-kvm`, and `virt-install` installed and running
- The `libvirtd` service active and enabled
- Python bindings for libvirt (`python3-libvirt`)
- Enough free disk space in the libvirt storage pool

On the **control node** (the machine running Ansible):

- Ansible 2.9 or later
- The `community.libvirt` collection:

```bash
ansible-galaxy collection install community.libvirt
```

## Installation

Clone the repository:

```bash
git clone https://github.com/YoucefElBoukhari/kvm-create-VMs-Lab.git
cd kvm-create-VMs-Lab
```

## Configuration

Default variables live in `roles/kvm_provision/defaults/main.yml`. Override them on the command line, in your inventory, or in a `vars` file.

> **Note:** The table below lists the typical variables for this kind of role. Adjust the names and values to match your actual `defaults/main.yml`.

| Variable            | Description                                  | Example                          |
|---------------------|----------------------------------------------|----------------------------------|
| `base_image_name`   | Filename of the base cloud image             | `Fedora-Cloud-Base-39.qcow2`     |
| `base_image_url`    | URL to download the base image               | `https://.../Fedora-Cloud...`    |
| `base_image_sha`    | Checksum to verify the downloaded image      | `sha256:...`                     |
| `libvirt_pool_dir`  | Directory of the libvirt storage pool        | `/var/lib/libvirt/images`        |
| `vm_name`           | Name of the virtual machine                  | `test-vm`                        |
| `vm_vcpus`          | Number of virtual CPUs                        | `2`                              |
| `vm_ram_mb`         | RAM in megabytes                             | `2048`                           |
| `vm_net`            | Libvirt network to attach to                 | `default`                        |
| `vm_root_pass`      | Root password for the VM                     | `changeme`                       |
| `ssh_key`           | Public SSH key injected into the VM          | `~/.ssh/id_ed25519.pub`          |
| `cleanup_tmp`       | Whether to remove temporary files after run  | `no`                             |

## Usage

Run the playbook against your KVM host. Provide the host either through an inventory file or with `--connection=local` if Ansible runs directly on the host:

```bash
ansible-playbook kvm_provision.yaml
```

Override variables at runtime to create a specific VM:

```bash
ansible-playbook kvm_provision.yaml \
  -e "vm_name=web-server vm_vcpus=4 vm_ram_mb=4096"
```

Run against a remote KVM host using an inventory:

```bash
ansible-playbook -i inventory kvm_provision.yaml
```

## How It Works

1. The role ensures the required packages and the libvirt service are present on the host.
2. It downloads the base cloud image (if not already cached) and verifies its checksum.
3. A dedicated qcow2 disk is created for the new VM from the base image.
4. The `vm-template.xml.j2` template is rendered with your variables to produce a libvirt domain definition.
5. The VM is defined and started via libvirt.

## Testing

A minimal test setup is included under `roles/kvm_provision/tests/`:

```bash
cd roles/kvm_provision/tests
ansible-playbook -i inventory test.yml
```

## Cleanup

To remove a VM created by this lab:

```bash
virsh destroy <vm_name>
virsh undefine <vm_name> --remove-all-storage
```

## License

Specify your license here (e.g. MIT, Apache-2.0, GPL-3.0).

## Author

**Youcef El Boukhari**
GitHub: [@YoucefElBoukhari](https://github.com/YoucefElBoukhari)

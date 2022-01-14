fast-vm-server
==============

This role configures OS for use with fast-vm and installs and configures the fast-vm on it.

Requirements
------------

This roles was tested on following systems and versions:
- AlmaLinux 8.5
- CentOS 7.9
- CentOS 8.5
- RHEL 7.8
- RHEL 8.1
- Fedora 31, 32, 33, 34, 35
- Debian 10.11 (not all features are supported)
- Debian 11.2 (not all features are supported)
- Ubuntu 18.04 (not all features are supported)
- Ubuntu 20.04 (not all features are supported)

On RHEL systems this role expects that system is properly registered so it can download and install packages.

(not all features are supported) - OVMF, fence_virtd and firewalld configuration was not tested and is disabled by default on some platforms.

Ansible 2.8.x or newer.

Role Variables
--------------

  - configure repositories needed for fast-vm installation
    ```
    config_repositories: true
    ```

  - install packages needed by fast-vm and fast-vm itself
    ```
    install_fastvm: true
    ```

  - configure libvirt for fast-vm access (change groups and permissions settings)
    ```
    config_libvirt_access: true
    ```

  - configure libvirt network for fast-vm (libvirtd service will be enabled after boot)
    ```
    config_libvirt_network: true
    ```

  - configure storage for fast-vm (create thinpool LV)
    ```
    config_storage: true
    ```

  - configure sudoers for fast-vm
    ```
    config_sudoers: true
    ```

  - generate /etc/fast.conf configuration file
    ```
    config_fastvm_conf: true
    ```

  - install OVMF UEFI firmware needed by UEFI fast-vm machines
    ```
    install_ovmf: true
    ```

  - install and configure fence_virtd that can be used to fence the fast-vm VMs using fence_xvm
    ```
    install_fence_virtd: true
    ```

  - install custom version of qemu-kvm,qemu-img and seabios-bin to support LSI and MEGASAS emaulated drivers
    ```
    install_custom_qemu: true
    ```

  - install and configure firewalld
    ```
    use_firewalld: true
    ```

  - group with access to fast-vm
    - **required by:** *config_libvirt_access, config_sudoers, config_fastvm_conf*
    ```
    fastvm_group: libvirt
    ```

  - name of VG where fast-vm thinpool LV is located
    - **required by:** *config_storage, config_fastvm_conf*
    ```
    fastvm_vg: c7vg
    ```

  - name of fast-vm thinpool LV
    - **required by:** *config_storage, config_fastvm_conf*
    ```
    fastvm_lv: fast-vm-pool
    ```

  - size of fast-vm thinpool LV
    - **required by:** *config_storage, config_fastvm_conf*
    ```
    fastvm_lv_size: 50G
    ```

  - fast-vm network subnet number
    - **required by:** *config_libvirt_network, config_fastvm_conf*
    ```
    fastvm_net: 42
    ```

  - name of fast-vm NAT libvirt network
    - **required by:** *config_libvirt_network, config_fastvm_conf, install_fence_virt*
    ```
    fastvm_net_name: fast-vm-nat
    ```

  - prefix for fast-vm VMs
    - **required by:** *config_fastvm_conf*
    ```
    fastvm_vm_prefix: 'fastvm-'
    ```

  - Allow only owners of VMs and 'root' to delete them
    - **required by:** *config_fastvm_conf*
    ```
    fastvm_owner_only_delete: 'yes'
    ```

  - Multicast address of fence_virt daemon
    - **required by:** *install_fence_virt*
    ```
    fence_virtd_address: '225.0.0.12'
    ```

  - Libguest appliance import or generation
    - **required by:** *config_fastvm_conf*
    ```
    fastvm_appliance: 'import'
    ```

  - System-wide default password for 'keydist' operation.
    - **required by:** *config_fastvm_conf*
    ```
    fastvm_keydist_password: 'testtest'
    ```

Example Playbook
----------------

Install and configure all basic things needed by fast-vm - default installation:

    - hosts: servers
      roles:
         - { role: ondrejhome.fast-vm-server }

Example hosts inventory file.

    [servers]
    el7-machine
    el8-machine
    fedora31-machine
    fedora32-machine
    fedora33-machine

License
-------

GPLv3

Author Information
------------------

To get in touch with author you can use email ondrej-xa2iel8u@famera.cz or create a issue on github when requesting feature(s).

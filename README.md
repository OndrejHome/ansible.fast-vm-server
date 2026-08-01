fast-vm-server
==============

This role configures OS for use with fast-vm, installs it, configures it and optionally install and configure additional services relevant to fast-vm.

Requirements
------------

This roles was tested on following systems and versions:
- AlmaLinux 8.10, 9.8, 10.2
- RHEL 8.10, 9.8, 10.2
- Fedora 42, 43, 44
- Debian 12.15, 13.6 (not all features are supported)
- Ubuntu 22.04.5, 24.04.3 (not all features are supported)

Tested with **Ansible 2.16.14**.

On RHEL systems this role expects that system is properly registered so it can download and install packages.

Role Variables
--------------

  - configure repositories needed for fast-vm installation
    ```
    config_repositories: true
    ```

  - install fast-vm and dependencies for playbook run
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

  - install ovmf-symlink package on systems where needed and check that OVMF firmware files exists in expected places
    ```
    install_ovmf: true
    ```

  - install and configure fence_virtd that can be used to fence the fast-vm VMs using fence_xvm
    ```
    install_fence_virtd: true
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
    fastvm_vg: a10vg
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

  - URL of fast-vm libguest appliance for importing
    NOTE: this will not overwrite existing appliance if there is a one in `/var/lib/fast-vm/appliance`
    ```
    fastvm_appliance_url: 'https://kr.famera.cz/fastvm-images/appliance-1.57.6-x86_64.tar.xz'
    ```

  - Force overwriting existing appliance (for example when trying to upgrade it)
    - `false` - do not overwrite appliance if it already exists
    - `true` - always overwrite appliance
    ```
    fastvm_appliance_force_import: false
    ```

  - System-wide default password for 'keydist' operation.
    - **required by:** *config_fastvm_conf*
    ```
    fastvm_keydist_password: 'testtest'
    ```


Example Playbook
----------------

**Example A:** Install and configure defaults on first VG on the system (good for system that have only one VG)

    - hosts: servers
      vars:
        fastvm_vg: "{{ ansible_lvm.vgs | first }}"
      roles:
        - { role: ondrejhome.fast-vm-server }


**Example B:** Use existing VG `vg_test` and allocate only 20GB for fast-vm LV on it with custom name `lv_for_vms`, leave rest on defaults. Note: If `vg_test/lv_for_vms` is an existing thinpool LV then this role will just use it and it will NOT recreate it.

    - hosts: servers
      vars:
        fastvm_vg: 'vg_test'
        fastvm_lv_size: '20G'
        fastvm_lv: 'lv_for_vms'
      roles:
        - { role: ondrejhome.fast-vm-server }


**Example C:** Create VG `vg_sdb` on disk `/dev/sdb` before installing and configuring defaults for fast-vm.

    - hosts: servers
      vars:
        fastvm_vg: "vg_sdb"
      roles:
        - { role: ondrejhome.fast-vm-server }
      pre_tasks:
        - name: create VG on /dev/sdb
          community.general.lvg:
            vg: "{{ fastvm_vg }}"
            pvs: '/dev/sdb'


**Example D:** Upgrade/Replace fast-vm libguest appliance only. Run this with `ansible-playbook -i hosts playbook.yaml --start-at-task='download libguestfs appliance into /tmp'` command.

    - hosts: servers
      vars:
        fastvm_appliance_url: 'https://kr.famera.cz/fastvm-images/appliance-1.57.6-x86_64.tar.xz'
        fastvm_appliance_force_import: true
      roles:
        - { role: ondrejhome.fast-vm-server }


Example hosts inventory file.

    [servers]
    el8-machine
    el9-machine
    fedora42-machine
    fedora43-machine
    fedora44-machine

License
-------

GPLv3

Author Information
------------------

To get in touch with author you can use email ondrej-xa2iel8u@famera.cz or create a issue on github when requesting feature(s).

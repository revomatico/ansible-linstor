# ansible-linstor

Ansible roles and sample playbook to deploy LINBIT's linstor on Linux (for now Ubuntu).

Inspired by [linstor-ansible](https://github.com/LINBIT/linstor-ansible).


## Features

- Deploys a fully functional linstor cluster
- Automatically creates storage pools
- Can use existing ETCD as database
- Deploys and configures CSI driver to Kubernetes
- No LINBIT account required as it does not depend on external resources


## Prerequisites

1. Python3 and pip3
2. Developed and tested with ansible 2.8, 2.9
3. Mitogen is highly recommended:
   - Download: `mkdir -p plugins && cd plugins && git clone https://github.com/dw/mitogen.git`
   - Set in **sample-absible.cfg**:
     - `strategy_plugins = plugins/mitogen/ansible_mitogen/plugins/strategy`
     - `strategy = mitogen_linear`

4. Setup ansible inventory (see [sample-inventory/hosts](sample-inventory/hosts))


## What is installed

If you want to use other specific versions, override `os_packages`:

### Ubuntu

1. `apt-cache search '(linstor*|drbd-*)' | awk '{print $1}' | xargs apt-cache madison | grep linbit-drbd9-stack | grep amd64`:

   ```
    drbd-dkms | 9.0.29-1ppa1~focal1 | http://ppa.launchpad.net/linbit/linbit-drbd9-stack/ubuntu focal/main amd64 Packages
   drbd-module-source | 9.0.29-1ppa1~focal1 | http://ppa.launchpad.net/linbit/linbit-drbd9-stack/ubuntu focal/main amd64 Packages
   drbd-reactor | 0.4.0-1ppa1~focal1 | http://ppa.launchpad.net/linbit/linbit-drbd9-stack/ubuntu focal/main amd64 Packages
   drbd-utils | 9.18.0-1ppa1~focal1 | http://ppa.launchpad.net/linbit/linbit-drbd9-stack/ubuntu focal/main amd64 Packages
   drbd8-utils | 2:9.18.0-1ppa1~focal1 | http://ppa.launchpad.net/linbit/linbit-drbd9-stack/ubuntu focal/main amd64 Packages
   linstor-client | 1.8.0-1ppa1~focal1 | http://ppa.launchpad.net/linbit/linbit-drbd9-stack/ubuntu focal/main amd64 Packages
   linstor-common | 1.13.0-1ppa1~focal1 | http://ppa.launchpad.net/linbit/linbit-drbd9-stack/ubuntu focal/main amd64 Packages
   linstor-controller | 1.13.0-1ppa1~focal1 | http://ppa.launchpad.net/linbit/linbit-drbd9-stack/ubuntu focal/main amd64 Packages
   linstor-satellite | 1.13.0-1ppa1~focal1 | http://ppa.launchpad.net/linbit/linbit-drbd9-stack/ubuntu focal/main amd64 Packages
   python-linstor | 1.8.0-1ppa1~focal1 | http://ppa.launchpad.net/linbit/linbit-drbd9-stack/ubuntu focal/main amd64 Packages

   ```

2. In `playbook-vars.yml` add (while keeping all the required packages in the list):

   ```yaml
   os_packages:
     ubuntu:
     - lvm2
     - drbd-dkms=9.0.25-1ppa1~focal1
     - drbd-utils=9.15.1-1ppa1~focal1
     - linstor-controller=1.10.0-1ppa1~focal1
     - linstor-satellite=1.10.0-1ppa1~focal1
     - linstor-client=1.4.2-1ppa1~focal1
   ```


## Run

- Full playbook:

    ```bash
    ANSIBLE_SSH_PIPELINING=true \
    ANSIBLE_CONFIG=sample-ansible.cfg \
    ansible-playbook sample-playbook.yml \
        --inventory sample-inventory \
        --extra-vars "@sample-vars.yml -v"
    ```

- Just software installation: add `--tags install`
- Just software removal: add `--tags uninstall`
- Just software configuration: add `--tags configure`
- Just storage pool creation: add `--tags storage`
- Just linstor-csi plugin: add `--tags csi`
- [DANGER] Destroy the cluster and wipe the disks: add `--tags destroy`. Data loss **will** occur!


## Role parameters

| Parameter                        | Default Value      | Description                                                                         |
|----------------------------------|--------------------|-------------------------------------------------------------------------------------|
| drbd_replication_network         | `undefined`        | Mandatory network for the replication mechanism. Format: a.b.c.d/mask               |
| linstor_controllers              | `undefined`        | Optional list of controllers to be manually specified instead of building automatically as a list of all controllers |
| linstor_pools                    | `undefined`        | A list of linstor pools to be created                                               |
| linstor_pools[].name             | `undefined`        | Name of the pool to be created                                                      |
| linstor_pools[].type             | `undefined`        | Pool type: filethin or lvmthin                                                      |
| linstor_pools[].size             | `100%FREE`         | Only for lvmthin. You can specify pool size in LVM format                           |
| linstor_pools[].physical_disks   | `undefined`        | Only for lvmthin. Comma separated disks OR a list of ansible inventory hostnames and physical devices |
| linstor_pools[].volume_group     | `undefined`        | Only for lvmthin. Overrides default volume group name (vg_name)                     |
| linstor_cotroller_endpoint       | `undefined`        | linstor controller endpoint URL for linstor-csi plugin to connect                   |
| etcd_env_file                    | `undefined`        | If specified, tries to load existing ETCD env file and use it as database and automagically sets the `linstor_db_*` parameters |
| linstor_db_connection_url        | `undefined`        | External database connection url (sets connection_url parameter in linstor.toml)    |
| linstor_db_user                  | `undefined`        | External database user                                                              |
| linstor_db_password              | `undefined`        | External database password                                                          |
| linstor_db_ca_certificate        | `undefined`        | External database CA certificate (To use with ETCD for example)                     |
| linstor_db_client_certificate    | `undefined`        | External database client certificate (To use with ETCD for example)                 |
| linstor_db_key                   | `undefined`        | External database client private key in PKCS8 format (To use with ETCD for example) |
| linstor_db_key_password          | `undefined`        | External database client private key password (To use with ETCD for example)        |
| vg_name                          | `linvg`            | Default volume group name for lvmthin                                               |
| filethin_base_path               | `/var/lib/linstor` | Default data directory for filethin                                                 |

## Samples

See [sample-vars.yml](sample-vars.yml)


## TODO
- [ ] Add more storage configuration options, like replicas, etc.

## Known issues
- If using etcd from preinstalled kubernetes and if this cluster was deployed with kubespray prior to 3.14 release, etcd vars will not be handled properly.
- Volume group "linstor_vg" has insufficient free space (5114 extents): 5120 required. This is because pool size is a bit too large (because of rounding).
    - Check vg free extents: `vgs --noheadings linstor_vg -o +vg_free_count | awk '{print $8; exit}'` (default extent size: 4MB).
    - Alternatively, do not specify size, 100%FREE will be used

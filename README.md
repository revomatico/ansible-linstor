!!!!!!!!!!!!!!!!!!!!!!!!!!!!!WIP!!!!!!!!!!!!!!!!!!!!!!!!!!!!

# ansible-linstor

Ansible role and sample playbook to deploy LINBIT's linstor on bare metal.

## Prerequisites
1. Python3 and pip3
2. Developed and tested with ansible 2.8
3. Mitogen is higly recommended:
  - Download: `mkdir -p plugins && cd plugins && git clone https://github.com/dw/mitogen.git`
  - Set in **sample-absible.cfg**:
    - `strategy_plugins = plugins/mitogen/ansible_mitogen/plugins/strategy`
    - `strategy = mitogen_linear`
4. Setup ansible inventory (see [sample-inventory/hosts](sample-inventory/hosts))

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
- Just storage pool creation: add `--tags storage`
- Just linstor-csi plugin: add `--tags csi`


## Role parameters

| Parameter                           | Default Value                | Description                                                           |
|-------------------------------------|------------------------------|-----------------------------------------------------------------------|
| drbd_replication_network            | `undefined`                  | Mandatory network for the replication mechanism. Format: a.b.c.d/mask |
| linstor_cotroller_ip                | `undefined`                  | linstor controller IP for linstor-csi plugin to connect               |


## Samples

> See [sample-vars.yml](sample-vars.yml):

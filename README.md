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
4. Setup ansible inventory:
    - To run remotely:
        - Set your ansible_host in `inventory/host_vars/master` to point to your kubernetes master
        - Set up your ssh key for authentication to the master
    - To run locally:
        - in `inventory/host_vars/master`, uncomment: `ansible_connection: local`
        - set the playbook variable `kubeconfig_file_path` pointing to a local kubeconfig


## Run
- Full playbook:

    ```bash
    ANSIBLE_SSH_PIPELINING=true \
    ANSIBLE_CONFIG=sample-ansible.cfg \
    ansible-playbook sample-playbook.yml \
        --inventory sample-inventory \
        --extra-vars "@params-override.yml -v"
    ```

- Just database creation: add `--tags data_clusters`
- Just simple schema creation: add `--tags create_schemas`
- Just SQL scripts: add `--tags execute_sql`


## Role parameters

| Parameter                           | Default Value                | Description                                                                                                                      |
|-------------------------------------|------------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| kubeconfig_file_path                | `undefined`                  | Optional path to kubeconfig file containing k8s cluster, user and context. The one in a default location will be used if empty   |
| kubernetes_retries                  | 10                           | k8s objects: Maximum number of retries before giving up in error and drama                                                       |
| kubernetes_delay                    | 15                           | k8s objects: Delay in seconds between retries                                                                                    |
| kubernetes_force                    | false                        | k8s objects: Replace the existing object instead of updating it                                                                  |
| kubernetes_state                    | `undefined`                  | k8s objects: If set to 'absent' will delete all stolon k8s objects, but the data should be left intact                           |
| stolon_namespace                    | stolon                       | Kubernetes namespace                                                                                                             |
| stolon_release                      | ''                           | Optional prefix for stolon objetcs                                                                                               |
| stolon_rbac                         | true                         | Enable RBAC                                                                                                                      |
| stolon_cluster                      | kube-stolon                  | Stolon cluster name. Will be prefixed with `stolon_release`-                                                                     |
| stolon_keeper_replicas              | 2                            | Number of keeper (statefulset) replicas                                                                                          |
| stolon_proxy_replicas               | 2                            | Number of proxy (deployment) replicas                                                                                            |
| stolon_sentinel_replicas            | 2                            | Number of sentinel (deployment) replicas                                                                                         |
| stolon_image                        | sorintlab/stolon:master-pg12 | Docker image for all stolon components                                                                                           |
| stolon_secret_password              | `generated`                  | Password for `stolon` database user. If not specified, a 15 length random password will be generated. Must NOT be base64 encoded |
| stolon_secret_replpassword          | `generated`                  | Password for the replication user. If not specified, a 15 length random password will be generated. Must NOT be base64 encoded   |
| |
| stolon_stkeeper_debug               | false                        | Enable debug for keeper                                                                                                          |
| stolon_stproxy_debug                | false                        | Enable debug for proxy                                                                                                           |
| stolon_stsentinel_debug             | false                        | Enable debug for sentinel                                                                                                        |
| |
| stolon_storage_class                | stolon-local-storage         | Name of k8s storage class to be created/used                                                                                     |
| stolon_storage_class_reclaim_policy | Retain                       | Reclaim policy, overriding k8s default of `Delete`                                                                               |
| |
| stolon_storage_size                 | 1Gi                          | Size of the local PersistentVolume                                                                                               |
| stolon_storage_local_path           | /stolon-local-data           | Data directory. PostgreSQL data will be in `stolon_storage_local_path`/postgres                                                  |
| |
| stolon_proxy_service.externalIPs    | `undefined`                  | Array of IPs for exposing proxy-service                                                                                          |
| stolon_proxy_service.port           | 5432                         | External proxy port                                                                                                              |
| |
| stolonctl_retries                   | 10                           | Number of retries for the stolonctl task before giving up. Increase this if hardware is slower                                   |
| stolonctl_delay                     | 15                           | Delay in seconds between each retry                                                                                              |
| |
| postgresql_schemas                  | []                           | List of objects to create roles with login and password and schemas                                                              |
| postgresql_scripts                  | []                           | List of SQL statements to run after database creation (e.g. DDL to create schemas, etc)                                          |


## Samples
> The following can be set in a vars file, e.g. `params-override.yml`

1. External IPs for the proxy service:
    ```yaml
    stolon_proxy_service:
      externalIPs:
      - 10.11.12.13
    ```

2. Create roles and schemas for applications
    ```yaml
    ## Automatically create schemas and roles (accounts) to login to the schemas for various apps
    postgresql_schemas:
    ## Create a schema keycloak, and set role and password the same:
    - name: "keycloak"
    ## Create a schema with a different role and password
    - name: "myschema"
      role: "myrole"
      password: "mypassword"
    ```

3. SQl Scripts:
    ```yaml
    postgresql_scripts:
    - |
      SOME OTHER SQL SCRIPT HERE;
    ```

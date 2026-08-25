# Ansible Molecule minimal working example with native configuration

The example "[Using podman containers](https://docs.ansible.com/projects/molecule/examples/podman/)" uses `ghcr.io/ansible/community-ansible-dev-tools:latest` as the container image. This minimal working example shows how to use other images.

## Observed behavior

Task `Wait for containers to be ready` fails with the message `timed out waiting for ping module test: 'ping'` but `podman ps` lists the container named `molecule-fedora`.

This can be reproduced using bb6ecbfa93b97b1584831e711cd130de2ab0d47b.

```
$ git rev-parse HEAD
bb6ecbfa93b97b1584831e711cd130de2ab0d47b
$ pixi run molecule test
INFO     default ➜ discovery: scenario test matrix: dependency, destroy, create, converge, idempotence, verify, cleanup, destroy
INFO     default ➜ prerun: Performing prerun with role_name_check=0...
INFO     default ➜ dependency: Executing
  ┌──────────────────────────────────────────────────────────────────────────────────
  │ ansible-galaxy install
  │ --role-file
  │   /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/requirements.yml
  │ 
  │ Starting galaxy collection install process
  │ Nothing to do. All requested collections are already installed. If you want to reinstall them, consider using
  │   `--force`.
  └─ Return code: 0 ─────────────────────────────────────────────────────────────────
INFO     default ➜ dependency: Dependency completed successfully.
  ┌──────────────────────────────────────────────────────────────────────────────────
  │ ansible-galaxy collection
  │   install
  │ --requirements-file
  │   /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/requirements.yml
  │ 
  │ Starting galaxy collection install process
  │ Nothing to do. All requested collections are already installed. If you want to reinstall them, consider using
  │   `--force`.
  └─ Return code: 0 ─────────────────────────────────────────────────────────────────
INFO     default ➜ dependency: Dependency completed successfully.
INFO     default ➜ dependency: Executed: Successful
INFO     default ➜ destroy: Executing
  ┌──────────────────────────────────────────────────────────────────────────────────
  │ ansible-playbook --inventory /home/raniere/.ansible/tmp/molecule.svhF.default/inventory
  │   --skip-tags molecule-notest,notest
  │ --inventory=inventory/
  │   /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/destroy.yml
  │ 
  │ 
  │ PLAY [Destroy container instances] *********************************************
  │ 
  │ TASK [Get info for all containers] *********************************************
  │ ok: [localhost] => (item=molecule-fedora)
  │ 
  │ TASK [Kill container if running] ***********************************************
  │ skipping: [localhost] => (item=molecule-fedora) 
  │ skipping: [localhost]
  │ 
  │ PLAY RECAP *********************************************************************
  │ localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
  │ 
  └─ Return code: 0 ─────────────────────────────────────────────────────────────────
INFO     default ➜ destroy: Executed: Successful
INFO     default ➜ create: Executing
  ┌──────────────────────────────────────────────────────────────────────────────────
  │ ansible-playbook --inventory /home/raniere/.ansible/tmp/molecule.svhF.default/inventory
  │   --skip-tags molecule-notest,notest
  │ --inventory=inventory/
  │   /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/create.yml
  │ 
  │ 
  │ PLAY [Create container instances] **********************************************
  │ 
  │ TASK [Create containers from inventory] ****************************************
  │ changed: [localhost] => (item=molecule-fedora)
  │ 
  │ TASK [Verify containers are running] *******************************************
  │ skipping: [localhost] => (item=molecule-fedora) 
  │ skipping: [localhost]
  │ 
  │ TASK [Wait for containers to be ready] *****************************************
  │ [WARNING]: Reset is not implemented for this connection
  │ [ERROR]: Task failed: Action failed: timed out waiting for ping module test: 'ping'
  │ Origin: /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/create.yml:31:7
  │ 
  │ 29         label: "{{ item.container.Name }}"
  │ 30
  │ 31     - name: Wait for containers to be ready
  │          ^ column 7
  │ 
  │ failed: [localhost -> molecule-fedora] (item=molecule-fedora) => {"ansible_loop_var": "item", "changed": false, "elapsed": 32, "item": "molecule-fedora",
  │   "msg": "timed out waiting for ping module test: 'ping'"}
  │ 
  │ PLAY RECAP *********************************************************************
  │ localhost                  : ok=1    changed=1    unreachable=0    failed=1    skipped=1    rescued=0    ignored=0
  │ 
  └─ Return code: 2 ─────────────────────────────────────────────────────────────────
CRITICAL Ansible return code was 2, command was: ansible-playbook --inventory /home/raniere/.ansible/tmp/molecule.svhF.default/inventory --skip-tags molecule-notest,notest --inventory=inventory/ /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/create.yml
ERROR    default ➜ create: Executed: Failed
WARNING  default ➜ create: An error occurred during the test sequence action: 'create'. Cleaning up.
INFO     default ➜ cleanup: Executing
  ┌──────────────────────────────────────────────────────────────────────────────────
  │ ansible-playbook --inventory /home/raniere/.ansible/tmp/molecule.svhF.default/inventory
  │   --skip-tags molecule-notest,notest
  │   --inventory=inventory/ /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/cleanup.yml
  │ 
  │ 
  │ PLAY [Cleanup container instances] *********************************************
  │ 
  │ TASK [Check if container is running] *******************************************
  │ ok: [molecule-fedora -> localhost]
  │ 
  │ TASK [Remove temporary files from running containers] **************************
  │ ok: [molecule-fedora]
  │ 
  │ PLAY RECAP *********************************************************************
  │ molecule-fedora            : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
  │ 
  └─ Return code: 0 ─────────────────────────────────────────────────────────────────
INFO     default ➜ cleanup: Executed: Successful
INFO     default ➜ destroy: Executing
  ┌──────────────────────────────────────────────────────────────────────────────────
  │ ansible-playbook --inventory /home/raniere/.ansible/tmp/molecule.svhF.default/inventory
  │   --skip-tags molecule-notest,notest
  │   --inventory=inventory/ /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/destroy.yml
  │ 
  │ 
  │ PLAY [Destroy container instances] *********************************************
  │ 
  │ TASK [Get info for all containers] *********************************************
  │ ok: [localhost] => (item=molecule-fedora)
  │ 
  │ TASK [Kill container if running] ***********************************************
  │ changed: [localhost] => (item=molecule-fedora)
  │ 
  │ PLAY RECAP *********************************************************************
  │ localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
  │ 
  └─ Return code: 0 ─────────────────────────────────────────────────────────────────
INFO     default ➜ destroy: Executed: Successful
INFO     default ➜ scenario: Pruning extra files from scenario ephemeral directory
ERROR    Ansible return code was 2, command was: ansible-playbook --inventory /home/raniere/.ansible/tmp/molecule.svhF.default/inventory --skip-tags molecule-notest,notest --inventory=inventory/ /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/create.yml
ERROR    Molecule executed 1 scenario (1 failed)

DETAILS                                                                        
default ➜ dependency: Executed: Successful
default ➜ destroy: Executed: Successful
default ➜ create: Executed: Failed (Ansible return code was 2, command was: ansible-playbook --inventory /home/raniere/.ansible/tmp/molecule.svhF.default/inventory --skip-tags molecule-notest,notest --inventory=inventory/ /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/create.yml)
default ➜ cleanup: Executed: Successful
default ➜ destroy: Executed: Successful

SCENARIO RECAP                                                                 
default                   : actions=5  successful=3  disabled=0  skipped=0  missing=0  failed=1
```

## Expected behavior

This can be reproduced using d14c9c6ea3699c66f9c005854bccf43b75563b92.

```
$ git rev-parse HEAD
d14c9c6ea3699c66f9c005854bccf43b75563b92
$ pixi run molecule test
INFO     default ➜ discovery: scenario test matrix: dependency, destroy, create, converge, idempotence, verify, cleanup, destroy
INFO     default ➜ prerun: Performing prerun with role_name_check=0...
INFO     default ➜ dependency: Executing
  ┌──────────────────────────────────────────────────────────────────────────────────
  │ ansible-galaxy install
  │ --role-file
  │   /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/requirements.yml
  │ 
  │ Starting galaxy collection install process
  │ Nothing to do. All requested collections are already installed. If you want to reinstall them, consider using
  │   `--force`.
  └─ Return code: 0 ─────────────────────────────────────────────────────────────────
INFO     default ➜ dependency: Dependency completed successfully.
  ┌──────────────────────────────────────────────────────────────────────────────────
  │ ansible-galaxy collection
  │   install
  │ --requirements-file
  │   /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/requirements.yml
  │ 
  │ Starting galaxy collection install process
  │ Nothing to do. All requested collections are already installed. If you want to reinstall them, consider using
  │   `--force`.
  └─ Return code: 0 ─────────────────────────────────────────────────────────────────
INFO     default ➜ dependency: Dependency completed successfully.
INFO     default ➜ dependency: Executed: Successful
INFO     default ➜ destroy: Executing
  ┌──────────────────────────────────────────────────────────────────────────────────
  │ ansible-playbook --inventory /home/raniere/.ansible/tmp/molecule.svhF.default/inventory
  │   --skip-tags molecule-notest,notest
  │ --inventory=inventory/
  │   /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/destroy.yml
  │ 
  │ 
  │ PLAY [Destroy container instances] *********************************************
  │ 
  │ TASK [Get info for all containers] *********************************************
  │ ok: [localhost] => (item=molecule-fedora)
  │ 
  │ TASK [Kill container if running] ***********************************************
  │ skipping: [localhost] => (item=molecule-fedora) 
  │ skipping: [localhost]
  │ 
  │ PLAY RECAP *********************************************************************
  │ localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
  │ 
  └─ Return code: 0 ─────────────────────────────────────────────────────────────────
INFO     default ➜ destroy: Executed: Successful
INFO     default ➜ create: Executing
  ┌──────────────────────────────────────────────────────────────────────────────────
  │ ansible-playbook --inventory /home/raniere/.ansible/tmp/molecule.svhF.default/inventory
  │   --skip-tags molecule-notest,notest
  │ --inventory=inventory/
  │   /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/create.yml
  │ 
  │ 
  │ PLAY [Create container instances] **********************************************
  │ 
  │ TASK [Create containers from inventory] ****************************************
  │ changed: [localhost] => (item=molecule-fedora)
  │ 
  │ TASK [Verify containers are running] *******************************************
  │ skipping: [localhost] => (item=molecule-fedora) 
  │ skipping: [localhost]
  │ 
  │ TASK [Wait for containers to be ready] *****************************************
  │ [WARNING]: Reset is not implemented for this connection
  │ ok: [localhost -> molecule-fedora] => (item=molecule-fedora)
  │ 
  │ PLAY RECAP *********************************************************************
  │ localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
  │ 
  └─ Return code: 0 ─────────────────────────────────────────────────────────────────
INFO     default ➜ create: Executed: Successful
INFO     default ➜ converge: Executing
  ┌──────────────────────────────────────────────────────────────────────────────────
  │ ansible-playbook --inventory /home/raniere/.ansible/tmp/molecule.svhF.default/inventory
  │   --skip-tags molecule-notest,notest
  │ --inventory=inventory/
  │   /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/converge.yml
  │ 
  │ 
  │ PLAY [Converge] ****************************************************************
  │ 
  │ TASK [Gathering Facts] *********************************************************
  │ ok: [molecule-fedora]
  │ 
  │ TASK [Read OS release information] *********************************************
  │ ok: [molecule-fedora]
  │ 
  │ TASK [Write OS info to file] ***************************************************
  │ changed: [molecule-fedora]
  │ 
  │ PLAY RECAP *********************************************************************
  │ molecule-fedora            : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
  │ 
  └─ Return code: 0 ─────────────────────────────────────────────────────────────────
INFO     default ➜ converge: Executed: Successful
INFO     default ➜ idempotence: Executing
  ┌──────────────────────────────────────────────────────────────────────────────────
  │ ansible-playbook --inventory /home/raniere/.ansible/tmp/molecule.svhF.default/inventory
  │   --skip-tags molecule-notest,notest,molecule-idempotence-notest
  │ --inventory=inventory/
  │   /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/converge.yml
  │ 
  │ 
  │ PLAY [Converge] ****************************************************************
  │ 
  │ TASK [Gathering Facts] *********************************************************
  │ ok: [molecule-fedora]
  │ 
  │ TASK [Read OS release information] *********************************************
  │ ok: [molecule-fedora]
  │ 
  │ TASK [Write OS info to file] ***************************************************
  │ ok: [molecule-fedora]
  │ 
  │ PLAY RECAP *********************************************************************
  │ molecule-fedora            : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
  │ 
  └─ Return code: 0 ─────────────────────────────────────────────────────────────────
INFO     default ➜ idempotence: Executed: Successful
INFO     default ➜ verify: Executing
  ┌──────────────────────────────────────────────────────────────────────────────────
  │ ansible-playbook --inventory /home/raniere/.ansible/tmp/molecule.svhF.default/inventory
  │   --skip-tags molecule-notest,notest
  │ --inventory=inventory/
  │   /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/verify.yml
  │ 
  │ 
  │ PLAY [Verify] ******************************************************************
  │ 
  │ TASK [Gathering Facts] *********************************************************
  │ ok: [molecule-fedora]
  │ 
  │ TASK [Read OS info file created during converge] *******************************
  │ ok: [molecule-fedora]
  │ 
  │ TASK [Verify OS is Fedora-based] ***********************************************
  │ ok: [molecule-fedora] => {
  │     "changed": false,
  │     "msg": "Successfully verified Fedora Linux-based container"
  │ }
  │ 
  │ TASK [Verify the dev tools environment variable] *******************************
  │ ok: [molecule-fedora] => {
  │     "changed": false,
  │     "msg": "Successfully verified ANSIBLE_DEV_TOOLS_CONTAINER"
  │ }
  │ 
  │ PLAY RECAP *********************************************************************
  │ molecule-fedora            : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
  │ 
  └─ Return code: 0 ─────────────────────────────────────────────────────────────────
INFO     default ➜ verify: Executed: Successful
INFO     default ➜ cleanup: Executing
  ┌──────────────────────────────────────────────────────────────────────────────────
  │ ansible-playbook --inventory /home/raniere/.ansible/tmp/molecule.svhF.default/inventory
  │   --skip-tags molecule-notest,notest
  │ --inventory=inventory/
  │   /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/cleanup.yml
  │ 
  │ 
  │ PLAY [Cleanup container instances] *********************************************
  │ 
  │ TASK [Check if container is running] *******************************************
  │ ok: [molecule-fedora -> localhost]
  │ 
  │ TASK [Remove temporary files from running containers] **************************
  │ changed: [molecule-fedora]
  │ 
  │ PLAY RECAP *********************************************************************
  │ molecule-fedora            : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
  │ 
  └─ Return code: 0 ─────────────────────────────────────────────────────────────────
INFO     default ➜ cleanup: Executed: Successful
INFO     default ➜ destroy: Executing
  ┌──────────────────────────────────────────────────────────────────────────────────
  │ ansible-playbook --inventory /home/raniere/.ansible/tmp/molecule.svhF.default/inventory
  │   --skip-tags molecule-notest,notest
  │ --inventory=inventory/
  │   /home/raniere/github.com/rgaiacs/ansible-molecule-mwe-native-configuration/molecule/default/destroy.yml
  │ 
  │ 
  │ PLAY [Destroy container instances] *********************************************
  │ 
  │ TASK [Get info for all containers] *********************************************
  │ ok: [localhost] => (item=molecule-fedora)
  │ 
  │ TASK [Kill container if running] ***********************************************
  │ changed: [localhost] => (item=molecule-fedora)
  │ 
  │ PLAY RECAP *********************************************************************
  │ localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
  │ 
  └─ Return code: 0 ─────────────────────────────────────────────────────────────────
INFO     default ➜ destroy: Executed: Successful
INFO     default ➜ scenario: Pruning extra files from scenario ephemeral directory
INFO     Molecule executed 1 scenario (1 successful)

DETAILS                                                                        
default ➜ dependency: Executed: Successful
default ➜ destroy: Executed: Successful
default ➜ create: Executed: Successful
default ➜ converge: Executed: Successful
default ➜ idempotence: Executed: Successful
default ➜ verify: Executed: Successful
default ➜ cleanup: Executed: Successful
default ➜ destroy: Executed: Successful

SCENARIO RECAP                                                                 
default                   : actions=8  successful=7  disabled=0  skipped=0  missing=0  failed=0
```
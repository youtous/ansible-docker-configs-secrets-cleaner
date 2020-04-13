# youtous/ansible-docker-configs-secrets-cleaner

This ansible role permits to easily remove docker stack's configs and secrets. 

```yaml
- name: docker prune secrets and configs
  include_role: youtous.docker-configs-secrets-cleaner
  vars:
    stack_name: "my_stack" # mandatory
    deployment_timestamp: "515155544554" # mandatory
```

### Usage

You should register a deployment timestamp using `{{ ansible_date_time.epoch }}`.
Secrets and configs names must be formated as `{{ stack_name }}_{{ anything you want }}-{{ deployment_timestamp }}`, for instance:

_docker-compose.yml_
```yaml
version: "3.7"

configs:
  whitelist_clients:
    name: "{{ stack_name }}_whitelist_clients-{{ deployment_timestamp }}"
    file: "./config/whitelist_clients.local"
```

### Discussion

- According the regex used `.*\s+{{ stack_name }}_.*-(?!{{ deployment_timestamp }})\d+.*\n` configs and secret without a timestamp will not be affected.
- Due to this bug https://github.com/ansible/ansible/issues/20493, this role cannot be used in handlers.

_todo : molecule tests and register on ansible-galaxy_

### Licence
MIT
# youtous/ansible-docker-configs-secrets-cleaner
[![pipeline status](https://gitlab.com/youtous/ansible-docker-configs-secrets-cleaner/badges/master/pipeline.svg)](https://gitlab.com/youtous/ansible-docker-configs-secrets-cleaner/-/commits/master)
[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white)](https://github.com/pre-commit/pre-commit)
[![GitHub Repo stars](https://img.shields.io/github/stars/youtous/ansible-docker-configs-secrets-cleaner?label=✨%20youtous%2Fansible-docker-configs-secrets-cleaner&style=social)](https://github.com/youtous/ansible-docker-configs-secrets-cleaner/)
[![Gitlab Repo](https://img.shields.io/badge/gitlab.com%2Fyoutous%2Fansible--docker--configs--secrets--cleaner?label=✨%20youtous%2Fansible-docker-configs-secrets-cleaner&style=social&logo=gitlab)](https://gitlab.com/youtous/ansible-docker-configs-secrets-cleaner/)
[![Licence](https://img.shields.io/github/license/youtous/ansible-docker-configs-secrets-cleaner)](https://github.com/youtous/ansible-docker-configs-secrets-cleaner/blob/master/LICENSE)


This ansible role permits to easily remove docker stack's configs and secrets.

After a successful deployment, you might want to delete old docker secrets/configs from your docker swarm cluster. The default behavior is to keep those files. This role makes deletion of those easily using a timestamp control on the resource's name.

Installation from [ansible galaxy](https://galaxy.ansible.com/youtous/docker_configs_secrets_cleaner): `ansible-galaxy install youtous.docker_configs_secrets_cleaner`

```yaml
- name: Docker prune secrets and configs
  ansible.builtin.include_role:
    name: youtous.docker_configs_secrets_cleaner
  vars:
    stack_name: "my_stack" # mandatory
    deployment_timestamp: "515155544554" # mandatory
```
### Requirements

- ansible-core 2.11+
- [community.docker](https://galaxy.ansible.com/community/docker) collection
- Packages that must be present on the system:
  - gawk
  - sed
  - grep

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

- According the regex used `'.*\s+{{ stack_name }}.*(\_|\-)(?!{{ deployment_timestamp }})\d+.*\n'` configs and secret without a timestamp will not be affected.
- Due to this bug https://github.com/ansible/ansible/issues/20493, this role cannot be used in handlers, a temporary workaround is:
```yaml
- name: Deploy traefik
  community.docker.docker_stack:
    state: present
    prune: true
    name: "{{ traefik_stack_name }}"
    compose:
      - "{{ traefik_compose_file_dest }}"
  notify: Docker prune
  register: stack_deploy_status

- name: Docker prune secrets and configs
  ansible.builtin.include_role:
    name: youtous.docker_configs_secrets_cleaner
  vars:
    stack_name: "{{ traefik_stack_name }}"
    deployment_timestamp: "{{ traefik_timestamp_deploy }}"
  when: stack_deploy_status.changed # noqa no-handler - until include_role is support in handlers...
```

### License

MIT

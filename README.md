# [Catena](https://github.com/alysoid/catena) Ansible Role: podman-systemd

Manage a rootless container orchestration with [Podman](https://wiki.archlinux.org/title/Podman) and systemd services.

## Role variables

### `podman_containers`

List of Podman containers to create or remove. The `state` can be `created` or `absent`. Containers are actually managed by systemd that removes them after they're stopped, so `started` and `present` falls into `created` and `stopped` falls into `absent`. When `started: yes` the container will be started via systemd. When `started: no` the container will be in status `created` and not running until the related systemd service will start.

```yaml
# Defaults
podman_containers: []

# Example
podman_containers:
  - name: nginx    # Podman container name
    state: created # (created|absent), default is `created`
    started: yes   # (yes|no), default is `yes`
  - name: whoami
    state: absent
    started: no
```

This role requires that a file named `{{ item.name }}.yml` will be present into the directory `{{ playbook_dir }}/compose/` for each element in the `podman_containers` list. In the example above you need to create two playbooks: `compose/whoami.yml` and `compose/nginx.yml`. Here's an example:

```yaml
# `container` is a helper that contains all the values for each element in `podman_containers` plus:
# `container.systemd`: contains the values in `podman_generate_systemd`
# `container.labels`: contains the values in `podman_container_labels`
- name: Service {{ container.name }}
  containers.podman.podman_container:
    name: "{{ container.name }}"
    state: "{{ container.state }}"
    generate_systemd: "{{ container.systemd }}"
    labels: "{{ container.labels }}"
    # https://hub.docker.com/_/nginx
    image: docker.io/library/nginx
    ports:
      - 8880:80
```

### `podman_autoupdate`

Auto-update containers via systemd timer/service:

```yaml
# Defaults
podman_autoupdate: yes
```

### `podman_autoupdate_timer`

Set `OnCalendar` value in time unit following [systemd.time](https://wiki.archlinux.org/title/Systemd/Timers) rules.

```yaml
# Defaults
podman_autoupdate_timer: "*-*-* 10:00:00"
```

### `podman_generate_systemd`

Options to generate systemd [unit file for containers](https://docs.ansible.com/ansible/latest/collections/containers/podman/podman_container_module.html).

[Podman auto-update](https://docs.podman.io/en/latest/markdown/podman-auto-update.1.html) expects that systemd units are generated with `new: yes`

```yaml
# Defaults
podman_generate_systemd:
  container_prefix: ""
  path: "{{ ansible_facts['user_dir'] }}/.config/systemd/user"
  restart_policy: always
  time: 10
  names: yes
  new: yes
```

### `podman_container_labels`

Define labels that will be applied to all containers.

```yaml
# Defaults
container_labels:
  # Enable auto-update policy
  io.containers.autoupdate: image
```

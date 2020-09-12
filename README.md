# Ansible Role: loki_canary

[![Build Status](https://travis-ci.org/patrickjahns/ansible-role-loki-canary.svg?branch=master)](https://travis-ci.org/patrickjahns/ansible-role-loki-canary)
[![License](https://img.shields.io/badge/license-MIT%20License-brightgreen.svg)](https://opensource.org/licenses/MIT)
[![Ansible Role](https://img.shields.io/badge/ansible%20role-patrickjahns.loki_canary-blue.svg)](https://galaxy.ansible.com/patrickjahns/loki_canary/)
[![GitHub tag](https://img.shields.io/github/tag/patrickjahns/ansible-role-loki-canary.svg)](https://github.com/patrickjahns/ansible-role-loki-canary/tags)

## Description

Deploy [loki canary](https://github.com/grafana/loki) using ansible.
For recent changes, please check the [CHANGELOG](/CHANGELOG.md) or have a look at [github releases](https://github.com/patrickjahns/ansible-role-loki-canary/releases)


## Requirements

- Ansible >= 2.7 

## Role Variables

All variables which can be overridden are stored in [defaults/main.yml](defaults/main.yml) file as well as in table below.

For configuration values related to canary itself, please refer to the [canary section](https://github.com/grafana/loki/blob/master/docs/operations/loki-canary.md#configuration) in the loki documentation

| Name                    | Default Value | Description                        |
| ----------------------- | ------------- | -----------------------------------|
| `canary_version`        | "1.6.1"       | canary package version. Also accepts *latest* as parameter. |
| `canary_system_user`    | loki-canary   | User the canary process will run at |
| `canary_system_group`   | "{{ canary_system_user }}" | Group of the *canary* user |
| `canary_install_dir`    | "/opt/loki-canary" | Directory where canary binaries will be installed |
| `canary_tmp_dir`        | "/tmp"        | Directory for temporary files during installation |
| `canary_config_addr`    | ""            | **REQUIRED** loki server address - [more information](https://github.com/grafana/loki/blob/master/docs/operations/loki-canary.md#configuration) |
| `canary_config_port`    | 3500          | Port where canary will expose metrics - [more information](https://github.com/grafana/loki/blob/master/docs/operations/loki-canary.md#configuration) |
| `canary_config_buckets` | ""            | [See canary configuration](https://github.com/grafana/loki/blob/master/docs/operations/loki-canary.md#configuration) |
| `canary_conig_interval` | ""            | [See canary configuration](https://github.com/grafana/loki/blob/master/docs/operations/loki-canary.md#configuration) |
| `canary_config_labelname` | ""          | [See canary configuration](https://github.com/grafana/loki/blob/master/docs/operations/loki-canary.md#configuration) |
| `canary_config_labelvalue` | ""         | [See canary configuration](https://github.com/grafana/loki/blob/master/docs/operations/loki-canary.md#configuration) |
| `canary_config_pruneinterval` | ""      | [See canary configuration](https://github.com/grafana/loki/blob/master/docs/operations/loki-canary.md#configuration) |
| `canary_config_size`    | ""            | [See canary configuration](https://github.com/grafana/loki/blob/master/docs/operations/loki-canary.md#configuration) |
| `canary_config_wait`    | ""            | [See canary configuration](https://github.com/grafana/loki/blob/master/docs/operations/loki-canary.md#configuration) |
| `canary_config_user`    | ""            | [See canary configuration](https://github.com/grafana/loki/blob/master/docs/operations/loki-canary.md#configuration) |
| `canary_config_pass`    | ""            | [See canary configuration](https://github.com/grafana/loki/blob/master/docs/operations/loki-canary.md#configuration) |
| `canary_redirect_stdout` | "False"      | Enable workaround to use canary with systemd |
| `canary_promtail_sd_file`   | "/etc/promtail/file_sd/canary.yml" | File service discovery file to write for promtail to discover |
| `canary_promtail_file_path` | "/tmp/canary.log" | Logfile to be written by canary and to be parsed by promtail |


## Role Tags

The following tags are specified in the role and can be selected/skipped during playbook runs

- `canary` - general tag that is attached to all tasks/handlers
- `canary_preflight` - attached to all preflight tasks (fetching shasums etc.)
- `canary_install` - attached to all install tasks
- `canary_run` - attached to the tasks that starts the canary service

## Example Playbook

Basic playbook that will assume that loki will be listening at `http://127.0.0.1:3100` 
```yaml
---
- hosts: all
  roles:
    - role: patrickjahns.loki_canary
      vars: 
        canary_config_addr: "http://127.0.0.1:3100"
```

More complex example, that configures [several parameters of loki canary](https://github.com/grafana/loki/blob/master/docs/operations/loki-canary.md#configuration)

```yaml
---
- hosts: all
  roles:
    - role: patrickjahns.loki_canary
      vars: 
        canary_config_addr: "http://127.0.0.1:3100"
        canary_config_port: 9000
        canary_config_wait: 3m
        canary_config_user: "test"
        canary_config_pass: "test"
        canary_config_labelname: "canary_host"
        canary_config_labelname: {{ ansible_hostname }}
```

Example to setup canary with systemd to [workaround upstream label issues](https://github.com/grafana/loki/issues/1435)

```yaml
---
- hosts: all
  roles:
    - role: patrickjahns.loki_canary
      vars: 
        canary_config_addr: "http://127.0.0.1:3100"
        canary_redirect_stdout: True
        canary_config_labelname: "canary_host"
        canary_config_labelvalue: "{{ ansible_hostname }}"
```

## Local Testing

The preferred way of locally testing the role is to use Docker and [molecule](https://github.com/metacloud/molecule) (v3.x). You will have to install Docker on your system. See "Get started" for a Docker package suitable to for your system.
We are using tox to simplify process of testing on multiple ansible versions. To install tox execute:
```sh
pip3 install tox
```
To run tests on all ansible versions (WARNING: this can take some time)
```sh
tox
```
To run a custom molecule command on custom environment with only default test scenario:
```sh
tox -e ansible29 -- molecule test -s default
```
For more information about molecule go to their [docs](http://molecule.readthedocs.io/en/latest/).

If you would like to run tests on remote docker host just specify `DOCKER_HOST` variable before running tox tests.

## CI

### Travis
Combining molecule and travis CI allows us to test how new PRs will behave when used with multiple ansible versions and multiple operating systems. This also allows use to create test scenarios for different role configurations. As a result we have a quite large test matrix which will take more time than local testing, so please be patient.

### Github Actions
Additionally to TravisCI some github actions are run to perform static code analysis

## License

This project is licensed under MIT License. See [LICENSE](/LICENSE) for more details.

## Maintainers and Contributors

- [Patrick Jahns](https://github.com/patrickjahns)
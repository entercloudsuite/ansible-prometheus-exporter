Ansible Role: prometheus-exporter 
======================================

[![Build Status](https://travis-ci.org/entercloudsuite/ansible-prometheus-exporter.svg?branch=master)](https://travis-ci.org/entercloudsuite/ansible-prometheus-exporter)
[![Galaxy](https://img.shields.io/badge/galaxy-entercloudsuite.prometheus-exporter-blue.svg?style=flat-square)](https://galaxy.ansible.com/entercloudsuite/prometheus-exporter)  

Installs prometheus-exporter on Ubuntu 16.04 (Xenial)

## Requirements

This role requires Ansible 2.4 or higher.

## Role Variables

The role defines most of its variables in `defaults/main.yml`:

## Example Playbook

Run with default vars:

    - hosts: all
      roles:
        - { role: ansible-prometheus-exporter }

## Testing

Tests are performed using [Molecule](http://molecule.readthedocs.org/en/latest/).

Install Molecule or use `docker-compose run --rm molecule` to run a local Docker container, based on the [enterclousuite/molecule](https://hub.docker.com/r/fminzoni/molecule/) project, from where you can use `molecule`.

1. Run `molecule create` to start the target Docker container on your local engine.  
2. Use `molecule login` to log in to the running container.  
3. Edit the role files.  
4. Add other required roles (external) in the molecule/default/requirements.yml file.  
5. Edit the molecule/default/playbook.yml.  
6. Define infra tests under the molecule/default/tests folder using the goos verifier.  
7. When ready, use `molecule converge` to run the Ansible Playbook and `molecule verify` to execute the test suite.  
Note that the converge process starts performing a syntax check of the role.  
Destroy the Docker container with the command `molecule destroy`.   

To run all the steps with just one command, run `molecule test`. 

In order to run the role targeting a VM, use the playbook_deploy.yml file for example with the following command: `ansible-playbook ansible-prometheus-exporter/molecule/default/playbook_deploy.yml -i VM_IP_OR_FQDN, -u ubuntu --private-key private.pem`.

# More example
## Deploy [node_exporter](https://github.com/prometheus/node_exporter)

### Playbook

simple
```yaml
- name: install node_exporter on all the instances
  hosts: all
  roles:
    - role: ansible-prometheus-exporter
      prometheus_exporter_name: node_exporter
```
more options
```yaml
- name: install node_exporter on all the instances
  hosts: all
  roles:
    - role: ansible-prometheus-exporter
      prometheus_exporter_name: node_exporter
      prometheus_exporter_config_flags:
        '--web.listen-address': '0.0.0.0:9100'
        '--log.level': 'info'
```


## Deploy [haproxy_exporter](https://github.com/prometheus/haproxy_exporter)


### Playbook

```yaml
    - role: entercloudsuite.prometheus-exporter
      prometheus_exporter_version: 0.9.0
      prometheus_exporter_name: "haproxy_exporter"
      prometheus_enable_exporter_config_flags: true
      prometheus_exporter_config_flags:
       '--haproxy.scrape-uri': 'unix:/run/haproxy/admin.sock'
```

### HAproxy configuration 

```
stats socket /run/haproxy/admin.sock mode 666 level admin
```

## Deploy [mysqld_exporter](https://github.com/prometheus/mysqld_exporter)

### Playbook

```yaml
  hosts: mysql_exporter
  roles:
    - role: entercloudsuite.prometheus-exporter
      prometheus_exporter_name: mysqld_exporter
      prometheus_exporter_version: 0.10.0
      prometheus_environment_variables:
        'DATA_SOURCE_NAME': 'exporter:v3rys3cr3tp4sw0rd@(mysqlhost:3306)/'
```

### mysql configuration

#### Creae user from monitoring
##### shell

```
CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'XXXXXXXX' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';
```

##### ansible task example mysql_user module

```yaml

  - name: Create user for monitoring
    mysql_user:
      name: "exporter"
      host: localhost
      password: "v3rys3cr3tp4sw0rd"
      priv: '*.*:PROCESS,REPLICATION CLIENT,SELECT'
      state: present

```


## Deploy [blackbox_exporter](https://github.com/prometheus/blackbox_exporter)
config file https://github.com/prometheus/blackbox_exporter/blob/master/blackbox.yml

Set **prometheus_exporter_custom_conf_destination** variable for deploy configuration file in a specific location  
```yaml
default-value: "{{ prometheus_exporters_common_root_dir }}/{{prometheus_exporter_name}}_current"
```

**prometheus_exporter_conf_main** config file location in playbook dir:  
### example:  
------------------------
```yaml
prometheus_exporter_conf_main: black_box_exporter_example_config.yaml
```
file location:
```BASH
$PLAYBOOKPATH/black_box_exporter_example_config.yaml
```
------------------------
```yaml
prometheus_exporter_conf_main: prometheus_cof/black_boxexporter/black_box_exporter_example_config.yaml
```
file location:
```BASH
$PLAYBOOKPATH/prometheus_cof/black_boxexporter/black_box_exporter_example_config.yaml
```

```yaml
prometheus_exporter_conf_main: black_box_exporter_example_config.yaml
```
### Playbook  
```yaml
- name: install blackbox_exporter on group
  hosts: blackbox_exporter
  roles:
    - role: ansible-prometheus-exporter
      prometheus_exporter_name: blackbox_exporter
      prometheus_exporter_version: 0.12.0
      # path to playbookpath/{{prometheus_exporter_conf_main}} custom path
      prometheus_exporter_conf_main: black_box_exporter_example_config.yaml
      prometheus_exporter_config_flags:
        "--config.file": "{{ prometheus_exporter_custom_conf_destination }}/black_box_exporter_example_config.yaml"

```

## Deploy [postgres_exporter](https://github.com/wrouesnel/postgres_exporter/)
```yaml
- name: install postgres_exporter on postgres_exporter group
  hosts: postgres_exporter
  roles:
    - role: ansible-prometheus-exporter
      prometheus_exporter_name: postgres_exporter
      url: https://github.com/wrouesnel/postgres_exporter/releases/download/v0.4.6/postgres_exporter_v0.4.6_linux-amd64.tar.gz
      prometheus_environment_variables:
        'DATA_SOURCE_NAME': 'postgresql://user:password@localhost:5432/?sslmode=disable'
```
## Deploy [uwsgi_exporter](https://github.com/AndreaGreco/prometeus_uwsgi_exporter)
```
- name: install uwsgi_exporter on uwsgi instance the instances
  hosts: uwsgi
  roles:
    - role: ansible-prometheus-exporter
      prometheus_exporter_name: uWSGI_expoter
      url: https://github.com/AndreaGreco/prometeus_uwsgi_exporter/files/1734745/uWSGI_expoter-v1.1.linux-amd64.tar.gz
      prometheus_exporter_conf_main: prometheus/config_uwsgi_expoter.yaml
      prometheus_exporter_config_flags:
        "-c": "{{ prometheus_exporters_common_root_dir }}/{{prometheus_exporter_name}}_current/config_uwsgi_expoter.yaml"
        "-n": ""
```

## License

MIT

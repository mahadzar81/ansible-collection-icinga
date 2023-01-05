# ansible-collection-icinga

## Overview

Collection to setup and manage components of the Icinga software stack. This collection is based on [Icinga2 Collection], plus additional roles icingaweb2 to install icingaweb2 and Icinga Director

## Requirements
None

## Usage

To use the collection in your playbooks, add the collection and then use the roles.

```
- name: Install icinga2 master
  hosts: all
  collections:
    - icinga.icinga
  gather_facts: true
  become: true

  vars:
    icinga2_constants:  # Set default constants and TicketSalt for the CA
      TicketSalt: "{{ lookup('ansible.builtin.password', '.icinga-server-ticketsalt') }}"
      NodeName: "{{ ansible_fqdn }}"
      ZoneName: "main"
    icinga2_confd: false # Disable example configuration
    icinga2_purge_features: true # Ansible will manage all features
    icinga2_config_directories:  # List of directories in which the role will manage monitoring objects
      - zones.d/main/commands
      - zones.d/main/hosts
      - zones.d/main/services
    icinga2_features:
      - name: api           # Enable Feature API
        ca_host: none       # No CA host, CA will be created locally
        endpoints:
          - name: NodeName
        zones:
          - name: ZoneName
            endpoints:
              - NodeName
              
      - name: checker
        state: present
      - name: notification
        state: present
      - name: idomysql
        state: present
        import_schema: false   # Enable this if the database and permissions for the user is created in advance.
        user: icinga2user
        password: <password>
        database: icinga2db
        cleanup:
          downtimehistory_age: 31d
          contactnotifications_age: 31d
      - name: mainlog
        severity: information

  roles:
    - { role: icinga.icinga.repos }
    - { role: icinga.icinga.icinga2 }
    - { role: icinga.icinga.icingaweb2 }
```

## License
MIT / BSD

[Icinga2 Collection]: https://github.com/Icinga/ansible-collection-icinga
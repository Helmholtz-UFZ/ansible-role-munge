Ansible Role: munge
=====================

An Ansible Role that installs and configures [MUNGE](https://github.com/dun/munge) 0.5.x

Table of Contents
-----------------

<!-- toc -->

- [Requirements](#requirements)
- [Role Variables](#role-variables)
  * [General](#general)
  * [munged Parameters](#munged-parameters)
- [Dependencies](#dependencies)
- [Example Playbook](#example-playbook)
  * [Top-Level Playbook](#top-level-playbook)
  * [Role Dependency](#role-dependency)
- [License](#license)
- [Author Information](#author-information)

<!-- tocstop -->

Requirements
------------

This Role utilizes the [synchronize](https://docs.ansible.com/ansible/latest/collections/ansible/posix/synchronize_module.html) (rsync) module that is available in Ansible to transfer the MUNGE-key from a central `munge_key_host` to all other Hosts that should form a Realm.

1. A passwordless SSH Authentication between the `munge_key_host` and the other Hosts is therefore required to make this Role work.
2. rsync must be installed on all hosts that MUNGE should be installed on.

Role Variables
--------------

### General

Available variables and their default values are explained below. Also see `defaults/main.yml`


```yml
munge_key_host: '{{ ansible_play_hosts_all | first }}'
```

This variable is very important. It specifies a central server on which the munge-key will be created (if it doesn't exist). Also, from this host the `rsync` command will be executed (via `synchronize` module) to safely transfer the key to the other Hosts of the Play that should form a MUNGE-Realm.
It is strongly recommeded to define this variable and specify a Host of your choice. Otherwise, it will always use the first Host in the Play as `munge_key_host`.


```yml
munge_service_enabled: yes
```

Controls whether the munge-service should be enabled on boot or not.


```yml
munge_enablerepo: ''
munge_disablerepo: ''
```

(Redhat/CentOS only) If you need to enable or disable any Repositories during installation. For both, this is a comma-seperated list, e.g: `epel,ius`

```yml
munge_packages:
  - munge
  - munge-libs
  - munge-devel
```

This variable per default holds a OS-specific list of packages (RedHat packages are here shown as example). You can use this variable to specify your own list of munge-packages that should be installed.

```yml
munge_user:
  name: munge
  group: munge
  comment: "Runs Uid 'N' Gid Emporium"
  shell: /sbin/nologin
  create_home: no
  system: yes
  # uid
  # gid
  # home
```

This variable can be used to create a self-defined user entry, for example, if you want a consistent uid/gid accross your Cluster for the munge User.

> Note: You shouldn't specify any other name/group than "munge" here for now. Currently, the service-files are not updated to start munged as the User and Group as defined per this variable.


```yml
munge_user_create: no
```

This controls whether the User entry defined in `munge_user` should be created prior to installing MUNGE. In most cases, it is not needed to configure your own user-entry.


### munged Parameters

For a more detailed explanation on these parameters, have a look at `man munged`. If a variable can be used depends on the Version of munge you have installed.

```yml
munge_syslog: no
```

If munged should log to `syslog` instead of the standard log-file. When yes, this applies the `--syslog` flag to munged and the `--log-file` flag will not be set.


```yml
munge_force: no
```

Whether munged should start despite several Warnings that may occur. Applies the `--force` flag.

```yml
munge_threads: 2
```

The amount of threads the munged should use. Applies the `--num-threads` option.

```yml
munge_verbose: no
```
Be verbose. Applies the `--verbose` option.


```yml
munge_origin_address: ''
```

Applies the `--origin` option. Should just be a single value, e.g. a hostname: `example.com`

```yml
munge_trusted_group: ''
```

Group Name or GID of a trusted group. Applies the `--trusted-group` option.

```yml
munge_max_ttl: 0
```

A value in seconds for max time-to-live for a credential. Applies the `--max-ttl` flag. With a value of `0`, this option will not be set (as `0` is a disallowed value for this option).

```yml
munge_mlockall: no
```

Controls munged page locking. Applies the `--mlockall` option.

```yml
munge_socket_file: /var/run/munge/munge.socket.2
```

(OS-specific, RedHat value as example value shown) This variable controls where the munged should have it's socket file. Since this value has an OS-specific value, it usually shouldn't be changed. If it's different from the OS default value, the `--socket` option is applied.

```yml
munge_pid_file: /var/run/munge/munged.pid
```

(OS-specific, RedHat value as example value shown) This variable controls where the munged should have it's PID file. Since this value has an OS-specific value, it usually shouldn't be changed. If it's different from the OS default value, the `--pid-file` option is applied.

```yml
munge_log_file: /var/log/munge/munged.log
```

(OS-specific, RedHat value as example value shown) This variable controls where the munged should have it's socket file. If it's different from the OS default value, the `--log-file` option is applied. However, if `munge_syslog: yes` is set, this option is ignored.

```yml
munge_seed_file: /var/lib/munge/munged.seed
```

(OS-specific, RedHat value as example value shown) This variable controls where the munged should have it's seed file. Since this value has an OS-specific value, it usually shouldn't be changed. If it's different from the OS default value, the `--seed-file` option is applied.

```yml
munge_key_file: /etc/munge/munge.key
```

(OS-specific, RedHat value as example value shown) This variable controls where the key-file is located that munged should use. Since this value has an OS-specific value, it usually shouldn't be changed. Applies the `--key-file` option (always). Note that this is also the location where the generated key is placed and where the `synchronize` module puts this file when transferred to the other hosts.


Dependencies
------------

None.

Example Playbook
----------------

Add to `requirements.yml`:

```yml
---

- src: ufz.munge

...
```

Download the role:

```console
$ ansible-galaxy install -r requirements.yml
```

### Top-Level Playbook

Write a top-level playbook:

```yml
---

- name: head server
  hosts: heads

  roles:
    - role: ufz.munge
      tags:
        - munge

...
```

### Role Dependency

Define the role dependency in `meta/main.yml`:

```yml
---

dependencies:

  - role: ufz.munge
    tags:
      - munge

...
```

License
-------

MIT

Author Information
------------------

This role was created by Toni Harzendorf (GitHub @tazend)

---
title: "ansible template 基本用法"
date: 2017-11-03T11:08:09+08:00
tags: ["ansible", "template"]
categories: ansible
slug: ansible-template-demo
type: post
---


```yaml
- name: config docker service
  template: 
    src: docker.service.j2
    dest: /etc/systemd/system/docker.service
    mode: 0644
  tags:
    - docker
```

会将当前模板下的 docker.service.j2 生成 /etc/systemd/system/docker.service 到对应的主机下

```shell
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=1min
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
```

全部用法如下

```
- name: Templates a file out to a remote server
  template:
      attributes:            # Attributes the file or directory should have. To get supported flags look at the man page for `chattr' on the target system. This string should contain the attributes in the same
                               order as the one displayed by `lsattr'.
      backup:                # Create a backup file including the timestamp information so you can get the original file back if you somehow clobbered it incorrectly.
      block_end_string:      # The string marking the end of a block.
      block_start_string:    # The string marking the beginning of a block.
      dest:                  # (required) Location to render the template to on the remote machine.
      follow:                # This flag indicates that filesystem links in the destination, if they exist, should be followed. Previous to Ansible 2.4, this was hardcoded as `yes'.
      force:                 # the default is `yes', which will replace the remote file when contents are different than the source.  If `no', the file will only be transferred if the destination does not
                               exist.
      group:                 # Name of the group that should own the file/directory, as would be fed to `chown'.
      mode:                  # Mode the file or directory should be. For those used to `/usr/bin/chmod' remember that modes are actually octal numbers (like 0644). Leaving off the leading zero will likely have
                               unexpected results. As of version 1.8, the mode may be specified as a symbolic mode (for example, `u+rwx' or `u=rw,g=r,o=r').
      newline_sequence:      # Specify the newline sequence to use for templating files.
      owner:                 # Name of the user that should own the file/directory, as would be fed to `chown'.
      selevel:               # Level part of the SELinux file context. This is the MLS/MCS attribute, sometimes known as the `range'. `_default' feature works as for `seuser'.
      serole:                # Role part of SELinux file context, `_default' feature works as for `seuser'.
      setype:                # Type part of SELinux file context, `_default' feature works as for `seuser'.
      seuser:                # User part of SELinux file context. Will default to system policy, if applicable. If set to `_default', it will use the `user' portion of the policy if available.
      src:                   # (required) Path of a Jinja2 formatted template on the Ansible controller. This can be a relative or absolute path.
      trim_blocks:           # If this is set to True the first newline after a block is removed (block, not variable tag!).
      unsafe_writes:         # Normally this module uses atomic operations to prevent data corruption or inconsistent reads from the target files, sometimes systems are configured or just broken in ways that
                               prevent this. One example are docker mounted files, they cannot be updated atomically and can only be done in an unsafe manner. This boolean option
                               allows ansible to fall back to unsafe methods of updating files for those cases in which you do not have any other choice. Be aware that this is
                               subject to race conditions and can lead to data corruption.
      validate:              # The validation command to run before copying into place. The path to the file to validate is passed in via '%s' which must be present as in the example below. The command is
                               passed securely so shell features like expansion and pipes won't work.
      variable_end_string:   # The string marking the end of a print statement.
      variable_start_string:   # The string marking the beginning of a print statement.
```
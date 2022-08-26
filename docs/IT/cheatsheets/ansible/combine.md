---

# Merge vars

**Combine** functionality to replace a specific value in a nested var

Global/all.yml:

```yaml
  sshd:
  Compression: true
  ListenAddress:
    - "0.0.0.0"
    - "::"
  PermitRootLogin: true
```

hosts_vars/hostname.yml
```yaml
  host_sshd:
  Compression: false
  PermitRootLogin: false
```

roles/sshd.yml

```yaml
- name: Combine default and host vars
  set_fact:
      sshd: "{{ sshd | combine(host_sshd) }}"
  when: host_sshd is defined

- name: "Configure sshd"
  include_role:
    name: willshersystems.sshd
```
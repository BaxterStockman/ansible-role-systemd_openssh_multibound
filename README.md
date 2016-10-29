# Ansible Role - systemd OpenSSH Multi-Bound

Ansible role for setting up
[socket-activated](https://www.freedesktop.org/software/systemd/man/systemd.socket.html)
OpenSSH daemons listening on more than one interface/port combination.  Useful
for running a locked-down daemon on a public address and a more lax daemon on
a private address.

## Requirements

Other than systemd and OpenSSH, nada.

## Role Variables

- `systemd_openssh_multibound_services`: A list of hashes of the form:
    - `name`: name associated with the systemd socket and service units.  For
      instance, if `name=sshd_lan`, this role will create `sshd_lan.socket` and
      `sshd_lan@.service`.  This parameter is mandatory.
    - `listen_streams`: a list in which each element is either a
      colon-separated string specifying host and port -- e.g.,
      `192.168.10.11:22` -- or a hash containing the keys `host` and `port`.
      Empty by default.  This parameter is mandatory.
    - `secure`: when true, `sshd` will start with options `PermitRootLogin=no`,
      `PasswordAuthentication=no`, and `LoginGraceTime=20s`.
    - `bind_to_device`: if present, `sshd` will bind only to this interface.

## Example Playbook

```yaml
- hosts: bastion
  roles:
    - role: BaxterStockman.systemd_openssh_multibound
      systemd_openssh_multibound_services:
        - bind_to_device: enp2s1
          name: sshd_wan
          secure: yes
          listen_streams:
            - host: 54.219.86.112
              port: 44829
            - port: 12312
            - 55.220.87.113:44830
        - name: sshd_lan
          secure: no
          listen_streams:
            - host: 192.168.10.10
              port: 22
            - 192.168.10.11:2222
```

## Notes

This role only creates `.socket` and corresponding `.service` files.  It does
not enable or start the `sshd` units, nor does it install the SSH daemon
program.

## License

MIT

## Author Information

Matt Schreiber - <mschreiber@gmail.com>

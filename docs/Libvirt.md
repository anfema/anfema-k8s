# Setting up `libvirt`

Make sure to have `libvirtd` configured and running:
1. Change the config files to contain the values of the two sections below
2. Make sure `libvirtd` is run with the `--listen` parameter or you run the socket activated systemd version.
  - If your system has it start and enable `libvirtd-tcp.socket`
  - If you do not run in socket activation mode edit the systemd unit file via
    `systemctl edit libvirtd`
  - If you distribution has an external config file (e.g.
    `/etc/conf.d/libvirtd`) preferrably use that one.
3. (Re-)start `libvirtd`: `systemctl restart libvirtd`
4. Create a sasl user to access the virtualization daemon remotely:
```bash
saslpasswd2 -a libvirt <user>
```
5. Now you can use `virt-manager` or `virsh` to connect to the VM cluster to
  monitor everything, use the connection URL `qemu+tcp://<hostname>:16509/system`
  or if you have ssh working `qemu+ssh://<user>@<hostname>/system` and install
  your ssh pubkey on `<hostname>`

To verify running VMs run `virsh -c <connection_url> list`, you will be prompted
for your username and password (the one from the sasl database)

To avoid specifying the URL all the time use this:

```bash
export VIRSH_DEFAULT_CONNECT_URI="<connection_url>"
```

## `/etc/libvirt/libvirtd.conf`

```
listen_tcp = 1
tcp_port = 16509
listen_addr = "0.0.0.0"

unix_sock_group = "libvirt"
unix_sock_ro_perms = "0777"
unix_sock_rw_perms = "0770"
unix_sock_admin_perms = "0700"
unix_sock_dir = "/run/libvirt"

auth_unix_ro = "none"
auth_unix_rw = "none"
auth_tcp = "sasl"
```

and put yourself in the `libvirt` group or connect via TCP, see config below.

## `/etc/sasl2/libvirt.conf`

```
mech_list: digest-md5
```

**Attention**: `DIGEST-MD5` is not secure for production systems, only use this
on testing systems! On newer Systems you can also use `scram-sha-1` which is
secure.
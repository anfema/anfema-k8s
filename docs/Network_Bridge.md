# Network bridge setup

For the cluster to be able to access the network you will have to setup a
network bridge to a network that can access the internet. Here's an example if
you're running `NetworkManager` (which most distributions do).

Reboot or restart NetworkManager after creating the following files.
You should see a new `br0` interface when running `ip addr` and the original
network interface should have no IP address anymore:

```
...
3: enp6s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master br0 state UP group default qlen 1000
    link/ether b4:2e:99:32:99:9b brd ff:ff:ff:ff:ff:ff
...
7: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 76:23:c1:a9:29:cb brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.155/24 brd 192.168.2.255 scope global dynamic noprefixroute br0
       valid_lft 36526sec preferred_lft 36526sec
...
```

## `/etc/NetworkManager/system-connections/bridge-br0.nmconnection`

This creates a new Network Bridge named `br0`

```
[connection]
id=bridge-br0
uuid=0285068e-c6e3-45ea-88d0-230d0963a963
type=bridge
interface-name=br0
permissions=

[bridge]
stp=false

[ipv4]
dns-search=
method=auto

[ipv6]
addr-gen-mode=stable-privacy
dns-search=
method=auto

[proxy]
```

## `/etc/NetworkManager/system-connections/bridge-slave-enp6s0.nmconnection`

This sets the network interface `enp6s0` to be a slave of the bridge.
Change `enp6s0` to the interface you use to connect to the Internet.

```
[connection]
id=bridge-slave-enp6s0
uuid=a2d946c6-663e-41ab-bb25-82eb1e65210a
type=ethernet
interface-name=enp6s0
master=br0
permissions=
slave-type=bridge

[ethernet]
mac-address-blacklist=

[bridge-port]
```

## Alternative setup with netplan (ubuntu)

Call the bridge 'br0' if you can.
Open /etc/netplan/50-cloud-init.yaml with your favorite editor

```
network:
    ethernets:
        enp1s0:
            dhcp4: true
    bridges:
        br0:
            dhcp4: true
            interfaces:
                - enp1s0
    version: 2
 ```

 Now run
 ```
 netplan apply
 ```

 Replace `enp1s0` with your ethernet device name.
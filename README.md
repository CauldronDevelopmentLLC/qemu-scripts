qemu-scripts
============
Shell scripts for managing qemu/kvm VMs.

# Usage

All of the scripts look for ``config.vm`` in the current directory.
``config.vm`` is a shell script which sets a number of environment variables
which effect how the scripts run.  At a minimum ``config.vm`` must contain
``ID=##`` where ``##`` is a positive integer which uniquely identifies the VM
in the host system.

 * startVM - Starts the VM.
 * connectVM - Opens a display to a running VM using ``spicy``.
 * mountVM - Mounts the directory ``mnt`` using to the root of a Windows VM
   using ``sshfs``.

# Setup

``startVM`` uses ``vde2`` for networking.  To configure first install the
following pacakges:

    sudo apt-get install -y vde2 dnsmasq qemu-kvm qemu-utils

Then add the following to ``/etc/network/interfaces``.

    auto vmtap
    iface vmtap inet static
      address 10.1.3.1
      netmask 255.255.255.0
      vde2-switch -t vmtap

Create ``/etc/dnsmasq.d/vm.conf`` with:

    interface=vmtap
    listen-address=10.1.3.1
    dhcp-range=vmtap,10.1.3.2,10.1.3.253,12h
    dhcp-host=02:00:00:00:00:00,10.1.3.10
    dhcp-host=02:00:00:00:00:01,10.1.3.11
    dhcp-host=02:00:00:00:00:02,10.1.3.12
    dhcp-host=02:00:00:00:00:03,10.1.3.13
    dhcp-host=02:00:00:00:00:04,10.1.3.14
    dhcp-host=02:00:00:00:00:05,10.1.3.15
    dhcp-host=02:00:00:00:00:06,10.1.3.16
    dhcp-host=02:00:00:00:00:07,10.1.3.17
    dhcp-host=02:00:00:00:00:08,10.1.3.18
    dhcp-host=02:00:00:00:00:09,10.1.3.19
    dhcp-host=02:00:00:00:00:10,10.1.3.20
    dhcp-host=02:00:00:00:00:11,10.1.3.21
    dhcp-host=02:00:00:00:00:12,10.1.3.22
    dhcp-host=02:00:00:00:00:13,10.1.3.23
    dhcp-host=02:00:00:00:00:14,10.1.3.24
    dhcp-host=02:00:00:00:00:15,10.1.3.25
    dhcp-host=02:00:00:00:00:16,10.1.3.26
    dhcp-host=02:00:00:00:00:17,10.1.3.27
    dhcp-host=02:00:00:00:00:18,10.1.3.28
    dhcp-host=02:00:00:00:00:19,10.1.3.29
    dhcp-host=02:00:00:00:00:20,10.1.3.30
    dhcp-host=02:00:00:00:00:21,10.1.3.31

Start ``vde2`` and ``dnsmasq``.

    sudo ifup vmtap
    sudo service dnsmasq start

# Allow VMs to access external network
You will need IP address masquerading to allow the VMs to access the external
network.  This can be accomplished with:

    sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    sudo iptables -A FORWARD -i eth0 -o vmtap -m state \
      --state ESTABLISHED,RELATED -j ACCEPT
    sudo iptables -A FORWARD -i vmtap -o eth0 -j ACCEPT

Where ``eth0`` is the external network interface.

To have this firewall configuration automatically restored on reboot install
iptables-persistent.

    sudo apt-get install -y iptables-persistent

Save the firewall rules with:

    sudo iptables-save | sudo tee /etc/iptables/rules.v4

Also make sure forwarding is enabled.

    sudo sysctl -w net.ipv4.ip_forward=1

To make this configuration permanent add ``net.ipv4.ip_forward = 1`` to
``/etc/sysctl.conf`` and save the firewall configuration.

# Linuxdeploy

* Deploy rootfs from OS. I used Ubuntu core
* Enable init sysv
* Add mount points:
  ```
  For setprop:
  /system
  setprop needs /apex/com.android.runtime/bin/linker64:
  /system/apex to /apex
  ```


# Termux
Get SSH server running
```
su

mount --bind /dev /path/to/chroot/environment/dev
mount --bind /dev/pts /path/to/chroot/environment/dev/pts
mount --bind /proc /path/to/chroot/environment/proc
mount --bind /sys /path/to/chroot/environment/sys

chroot /data/linux

/bin/su -

# maybe usermod -g aid_inet _apt

apt update
apt install openssh-server --no-install-recommends
```

# Webserver (optional)
append user www-data to group aid_inet for socket creation

```
usermod -aG aid_inet www-data
```


# USB Tethering

```
# switch usb to rndis mode
setprop sys.usb.config rndis

# set ip and enable interface
ip address add 192.168.0.1/24 dev rndis0
ip link set dev rndis0 up

# enable forwarding (without this no packets between interfacs)
echo 1 > /proc/sys/net/ipv4/ip_forward

# setup iptables
update-alternatives --set iptables /usr/sbin/iptables-legacy
iptables -F
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
# block incoming traffic for local machine
iptables -A INPUT -i rmnet_data0 -j DROP
# forward all incoming traffic to router
iptables -A PREROUTING -i rmnet_data0 -j DNAT --to-destination 192.168.0.2
# iptables -t nat -A POSTROUTING -o rmnet_data0 -j MASQUERADE

# add main table to rules
ip rule add from all lookup main

# without this packets don't reach rndis0 (they still reach rmnet_data0)
# this was created automatically by OS
# ip route add 192.168.0.0/24 dev rndis0 table main

# without this packets don't reach rmnet_data0 (they still reach rndis0)
ip route add default via <rmnet_data0 IP> dev rmnet_data0 table main
```

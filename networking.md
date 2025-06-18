# commands

`nmcli connection show`\
`nmcli connection up <>`\
`nmcli connection down <>`

`nmcli connection delete "<>"`\
`nmcli connection add tupe ethernet ifname enp0s3 con-name "nat" ipv4.method auto`

# Create new Network adapters in oracle virtual box

Tools > Create (Remember to enable DHCP server for it after creating)

# Problem
## i installed a machine with attached to NAT network adapter but when i start the machine the NAT network adapter got wrong ip and i had no internet access:

*so first i used command :*

`nmcli connection show`

*to see what is the name of that connection name then i used command :*

`nmcli connection delete \<the connection name\>`

*to delete that connection*

*then the command :*

`nmcli connection add tupe ethernet ifname enp0s3 con-name "nat" ipv4.method auto`

*to create a new connection named nat for interface `enp0s3` and give it ip automatically*

*after that the ip of that interface was correct and i had internet access*

______________________________

***i had another problem that when my vm started the adapter couldnt get ip from dhcp server of virtualbox network (it was host only)***

*then i should give it ip manually*

*so i used commands below:*

`nmcli connection show` *to see connections list*\
`nmcli connection delete enp0s3` *to delete the connection which dosnt have ip (name of connection was enp0s3)*\
`nmcli connection add type ethernet ifname enp0s3 ipv4.addresses 192.168.12.4/24 ipv4.gateway 192.168.12.1 ipv4.dns "8.8.8.8,8.8.4.4" ipv4.method manual` *to give ip to connection manually*


__________________________________

## another problem : NetworkManager uninstalled

**Problem:** *yesterday i tried to install java package on 2 rocky machines which had no internet access. so i downloaded the packages related to Java on another rocky machine which had internet access using command below:*
```
sudo dnf download --resolve --alldeps java-17-openjdk
```
*then i sent all these downloaded packages to two offline rocky machines and used the command below to install Java on these machines :*
```
sudo dnf install *.rpm --disablerepo=* --nogpgcheck --allowerasing
```
*The `--allowerasing` option tells DNF:*\
*“If there are conflicts between what I'm installing and what's already installed, go ahead and remove the conflicting packages to make it work.”*\
*So if your offline RPM installation included packages that had conflicts with `NetworkManager`, or needed other networking-related packages that conflicted, DNF may have erased `NetworkManager` to satisfy dependencies.*

so the `NetworkManager` is uninstalled on machines. then after a restart the interface had no ip address. and o couldn't use `nmcli` command to reconfigure it. then how i solved the problem?

**Answer :**\

You can do this with `ip` or `ifconfig`:

1. **Check interface name**
```
ip link
```
Look for something like enp0s3, eth0, etc.
2. **Assign IP address**
```
sudo ip addr add 192.168.12.4/24 dev enp0s8
```
3. **Bring up the interface**
```
sudo ip link set enp0s8 up
```
4. **Set default gateway**
```
sudo ip route add default via 192.168.12.1
```
5. **Set DNS (edit resolv.conf)**
```
sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'
```

after these steps my problem solved.



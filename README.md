# K4YT3X's Hardened sysctl Configuration

This repository hosts my hardened version of `sysctl.conf`. This configuration file aims to provide better security for Linux systems, and improves system performance whenever possible. For example, below are some of the features this configuration file provies.

- Prevents kernel pointers from being read
- Disables Ptrace for all programs
- Disallows core dumping by SUID/GUID programs
- Disables IPv4/IPv6 routing
- Enables BBR TCP congestion control
- Enables SYN cookies to mitigate SYN flooding attacks
- Enables IP reverse path filtering for source validation
- ...

**Please review the configuration file carefully before applying it.** You are responsible for actions done to your own system. If you need some guidance understanding what each of the settings are for, [sysctl-explorer](https://sysctl-explorer.net/) might come in handy.

Please be careful that this `sysctl.conf` is **designed for endpoint hosts that do not act as a router**. If you would like to use this configuration file on a router, please go over the configuration file and make necessary changes.

## Usages

1. Download the file `sysctl.conf` from the repository
1. **Review the content of the `sysctl.conf` file to make sure all settings are suitable for your system**
1. Backup your current `/etc/sysctl.conf` file (e.g., `cp /etc/sysctl.conf /etc/sysctl.conf.backup`)
1. Overwrite the old `sysctl.conf` file with the downloaded `sysctl.conf` file
1. Run command `sudo sysctl -p` or reboot the system to apply the changes

```shell
# clone the repository
git clone https://github.com/k4yt3x/sysctl.git ~/sysctl

# backup the original sysctl.conf
sudo cp /etc/sysctl.conf /etc/sysctl.conf.backup

# replace the old sysctl.conf with the new one
sudo cp ~/sysctl/sysctl.conf /etc/sysctl.conf

# apply changes
sudo sysctl -p

# remove the downloaded repository if you don't need it anymore
rm -rf ~/sysctl
```

For convenience, I have pointed the URL `https://akas.io/sysctl` to the `sysctl.conf` file. You may therefore download the `sysctl.conf` file with the following command. However, be sure to check the integrity of the file after downloading it if you choose to download using this method.

```shell
curl -sSL akas.io/sysctl -o sysctl.conf
```

## `sysctl.conf` Content

```properties
# Name: K4YT3X Hardened sysctl Configuration
# Author: K4YT3X
# Date Created: October 5, 2020
# Last Updated: October 5, 2020
# Version: 1.0

# Licensed under the GNU General Public License Version 3 (GNU GPL v3),
#   available at: https://www.gnu.org/licenses/gpl-3.0.txt
# (C) 2020 K4YT3X

# Multiple sources have been consulted while writing this configuration
#  file (e.g., nixCraft's sysctl.conf). Sources are not cited since this
#  is not an academic document. Please refer to Linux documentations
#  should you have any questions.

########## Kernel ##########

# enable ExecShield protection
# 2 enables ExecShield by default unless applications bits are set to disabled
# uncomment on systems without NX/XD protections
# check with: dmesg | grep --color '[NX|DX]*protection'
#kernel.exec-shield = 2

# enable ASLR
# turn on protection and randomize stack, vdso page and mmap + randomize brk base address
kernel.randomize_va_space = 2

# controls the System Request debugging functionality of the kernel
kernel.sysrq = 0

# controls whether core dumps will append the PID to the core filename
# useful for debugging multi-threaded applications
kernel.core_uses_pid = 1

# restrict access to kernel address
# kernel pointers printed using %pK will be replaced with 0’s regardless of privileges
kernel.kptr_restrict = 2

# Ptrace protection using Yama
#   - 1: only a parent process can be debugged
#   - 2: only admins canuse ptrace (CAP_SYS_PTRACE capability required)
#   - 3: disables ptrace completely, reboot is required to re-enable ptrace
kernel.yama.ptrace_scope = 3

# allow for more PIDs
kernel.pid_max = 65536

# reboot machine after kernel panic
#kernel.panic = 10

########## File System ##########

# disallow core dumping by SUID/SGID programs
fs.suid_dumpable = 0

# protect the creation of hard links
# one of the following conditions must be fulfilled
#   - the user can only link to files that he or she owns
#   - the user must first have read and write access to a file, that he/she wants to link to
fs.protected_hardlinks = 1

# protect the creation of symbolic links
# one of the following conditions must be fulfilled
#   - the process following the symbolic link is the owner of the symbolic link
#   - the owner of the directory is also the owner of the symbolic link
fs.protected_symlinks = 1

# increase system file descriptor limit
fs.file-max = 65535

########## IPv4 Networking ##########

# enable BBR congestion control
net.ipv4.tcp_congestion_control = bbr

# disallow IPv4 packet forwarding
net.ipv4.ip_forward = 0

# enable SYN cookies for SYN flooding protection
net.ipv4.tcp_syncookies = 1

# number of times SYNACKs for a passive TCP connection attempt will be retransmitted
net.ipv4.tcp_synack_retries = 5

# do not send redirects
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.send_redirects = 0

# do not accept packets with SRR option
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.accept_source_route = 0

# enable reverse path source validation
# refer to RFC1812
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.rp_filter = 1

# log packets with impossible addresses to kernel log
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1

# do not accept ICMP redirect messages
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.secure_redirects = 0
net.ipv4.conf.all.secure_redirects = 0

# disable sending and receiving of shared media redirects
# this setting overwrites net.ipv4.conf.all.secure_redirects
# refer to RFC1620
net.ipv4.conf.all.shared_media = 0

# ignore all ICMP ECHO and TIMESTAMP requests sent to broadcast/multicast
net.ipv4.icmp_echo_ignore_broadcasts = 1

# ignore bad ICMP errors
net.ipv4.icmp_ignore_bogus_error_responses = 1

# always use the best local address for announcing local IP via ARP
net.ipv4.conf.all.arp_announce = 2

# reply only if the target IP address is local address configured on the incoming interface
net.ipv4.conf.all.arp_ignore = 1

# mitigate TIME-WAIT Assassination hazards in TCP
# refer to RFC1337
net.ipv4.tcp_rfc1337 = 1

# disable TCP window scaling
# this makes the host less susceptible to TCP RST DoS attacks
net.ipv4.tcp_window_scaling = 0

# increase system IP port limits
net.ipv4.ip_local_port_range = 2000 65000

# disable TCP timestamps for better CPU utilization
net.ipv4.tcp_timestamps = 0

# enable TCP selective ACKs for better throughput
net.ipv4.tcp_sack = 1

# divide socket buffer evenly between TCP window size and application
net.ipv4.tcp_adv_win_scale = 1

# increase the maximum length of processor input queues
net.core.netdev_max_backlog = 250000

# increase memory thresholds to prevent packet dropping
#net.ipv4.tcp_rmem = 4096 87380 8388608
#net.ipv4.tcp_wmem = 4096 87380 8388608

# increase TCP max buffer size setable using setsockopt()
#net.core.rmem_max = 8388608
#net.core.wmem_max = 8388608
#net.core.rmem_default = 8388608
#net.core.wmem_default = 8388608
#net.core.optmem_max = 8388608

########## IPv6 Networking ##########

# disallow IPv6 packet forwarding
net.ipv6.conf.all.forwarding = 0

# number of Router Solicitations to send until assuming no routers are present
net.ipv6.conf.default.router_solicitations = 0

# do not accept Router Preference from RA
net.ipv6.conf.default.accept_ra_rtr_pref = 0

# learn prefix information in router advertisement
net.ipv6.conf.default.accept_ra_pinfo = 0

# setting controls whether the system will accept Hop Limit settings from a router advertisement
net.ipv6.conf.default.accept_ra_defrtr = 0

# router advertisements can cause the system to assign a global unicast address to an interface
net.ipv6.conf.default.autoconf = 0

# number of neighbor solicitations to send out per address
net.ipv6.conf.default.dad_transmits = 0

# number of global unicast IPv6 addresses can be assigned to each interface
net.ipv6.conf.default.max_addresses = 1
```

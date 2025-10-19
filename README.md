# K4YT3X's Hardened Linux Kernel Parameters

> [!IMPORTANT]
> This documentation contains important information. It is highly recommended that you read through the entire document and do your own research to fully understand the implications of applying this configuration to your own system.

> [!TIP]
> Don't forget to apply the boot command line options at the end of this page in addition to the sysctl configuration file.

This repository hosts an **aggressively** hardened version of `sysctl.conf`. This configuration file aims to provide better security for Linux systems and to improve system performance wherever possible. Below are some of the features this configuration file provides:

- Prevents kernel pointers from being read
- Disables ptrace for all programs
- Disallows core dumping by SUID/SGID programs
- Disables IPv4/IPv6 routing
- Enables BBR TCP congestion control
- Enables SYN cookies to mitigate SYN flooding attacks
- Enables IP reverse path filtering for source validation
- …

**Please review the configuration file carefully before applying it.** You are ultimately responsible for your own system. If you need some guidance understanding what each setting is for, the [sysctl-explorer](https://sysctl-explorer.net/) might come in handy. You may also find [Linux's kernel documentation](https://www.kernel.org/doc/Documentation/sysctl/) useful.

## Assumptions

This configuration file is written with a few assumptions about your system. You can still use this configuration as a template if your system does not match these assumptions (e.g., set `net.ipv4.ip_forward` to `1` if your system also acts as a router). Making these assumptions helps in developing a configuration file that enables as many optimizations as possible for common systems.

- Security is valued over performance and convenience
- Your system does not act as a router
- You have a 64-bit system
- Your system is on a network that is relatively stable (e.g., wired Ethernet)
- No debugging features are required (e.g., GDB/kdump)
- ICMP echo and echo reply messages are not considered dangerous

## Configuration Basics

Linux kernel runtime configuration files are typically stored in the `/etc/sysctl.d` directory, where all `.conf` files are automatically loaded by a service like `systemd-sysctl`. The exact file locations and loading behavior may vary depending on the distribution and the sysctl service in use.

Files are sorted and read by their file names in lexicographic order. **Variables read later will overwrite variables read earlier.** For example, configurations in `20-something.conf` will be read before `99-sysctl.conf`. If the same variable exists in both files, values read from `20-something.conf` will be overwritten by values read from `99-sysctl.conf`:

```shell
$ cat /etc/sysctl.d/20-something.conf
net.ipv4.ip_forward = 0

$ cat /etc/sysctl.d/99-sysctl.conf
net.ipv4.ip_forward = 1

$ sysctl -a | grep net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```

Also, sysctl services might load configuration files from multiple paths. For example, the `systemd-sysctl` service also discovers configuration files from these directories in addition to `/etc/sysctl.d`:

- `/run/sysctl.d`
- `/usr/local/lib/sysctl.d`
- `/usr/lib/sysctl.d`
- `/lib/sysctl.d`

It is a good idea to verify that no other sysctl configuration files are being loaded, to ensure that your settings are not overridden by values in another file. On a system using systemd, you can check which configuration files are loaded and in what order by running:

```shell
systemd-analyze cat-config sysctl
```

## Recommended Deployment Strategy

I recommend deploying this configuration file as a template, then add your custom override values in another configuration file that will be loaded after the template file (e.g., `/etc/sysctl.d/99-sysctl.conf`). For example, I personally have this template deployed to `/etc/sysctl.d/98-k4yt3x.conf`, then added several override values in `/etc/sysctl.d/99-sysctl.conf`.

The advantage of this approach is ease of maintenance. The same template configuration file can be deployed across multiple machines, while machine-specific customizations are kept in a separate file. This way, you can upgrade the template as new versions are released without having to migrate the customized values. It also makes it easier to see which customizations are applied to a given machine. I will walk you through the deployment steps below.

First, we need to download the template configuration file from this repository:

```shell
sudo curl https://raw.githubusercontent.com/k4yt3x/sysctl/master/sysctl.conf -o /etc/sysctl.d/98-k4yt3x.conf
```

Then, you can add your custom values to a configuration file that will be loaded after the template configuration file, such as `/etc/sysctl.d/99-sysctl.conf`. Here are some custom overrides I have added to one of my workstations for convenience and performance:

```ini
# Allow debuggers like GDB to ptrace its descendants
kernel.yama.ptrace_scope = 1

# Enable TCP timestamps with RFC 1323 randomized offsets
net.ipv4.tcp_timestamps = 1

# Enable SACK to assist congestion control
net.ipv4.tcp_sack = 1
net.ipv4.tcp_dsack = 1
net.ipv4.tcp_fack = 1
```

Be aware that values from the template may be overwritten by other configuration files. For example, on my system a file named `uhd-usrp2.conf` is loaded after `99-sysctl.conf` and overrides the values of `net.core.rmem_max` and `net.core.wmem_max` defined earlier. Package managers can add new configuration files when you install or update packages, so you need to be careful that your custom settings are not overridden by those files.

## Loading and Verifying the Changes

For the changes to be effective, you will have to reload the sysctl configurations. This can be achieved by either rebooting your machine or reloading the configurations using one of the following commands:

```shell
# Instruct sysctl to load settings from the configuration files into the live kernel
# This command allows you to see the variables as they are being loaded
sudo sysctl --system

# Alternatively, you can restart the systemd-sysctl service on a system that uses systemd
sudo systemctl restart systemd-sysctl
```

Afterwards, verify your changes by dumping live kernel parameters. Replace `your.config` in the following command with the name of the variable you would like to check:

```shell
sysctl your.config
```

For example, the following command prints the value of `kernel.kptr_restrict`:

```shell
$ sysctl kernel.kptr_restrict
kernel.kptr_restrict = 2
```

## Boot Command Line

In addition to the sysctl configuration file, which sets kernel parameters at runtime, some parameters have to be set at boot time via the boot command line. Below are some recommended options that can be set in your bootloader configuration (e.g., GRUB or systemd-boot) to harden your system further:

```ini
# Treat kernel oops as a panic
ops=panic

# Let the kernel panic on correctable MCE errors
mce=0

# Disable merging of slabs of similar sizes
slab_nomerge

# Fill freed and allocated pages/heap objects with zeros
init_on_free=1
init_on_alloc=1

# Enable page poisoning and page ownership tracking
page_poison=1
page_owner=1

# Enable SLUB sanity checks, red zoning, and poisoning
slub_debug=FZP

# Perform usercopy bounds checking
hardened_usercopy=1

# Force exposed pointers to be hashed
hash_pointers=always

# Enable page allocator freelist randomization
page_alloc.shuffle=1

# Randomize kernel stack offset on syscall entry
randomize_kstack_offset=on

# Disable speculative store bypass
spec_store_bypass_disable=on

# Enable IOMMU for Intel and AMD systems
intel_iommu=on
amd_iommu=on

# Force IOMMU TLB invalidation
iommu.passthrough=0
iommu.strict=1

# Disable virtual syscalls
vsyscall=none

# Disable the 32-bit vDSO
vdso32=0

# Disable FineIBT since it is weaker than pure KCFI
cfi=kcfi

# Mitigates all known CPU vulnerabilities
mitigations=auto,nosmt

# Enable MDS mitigations and disable SMT
mds=full,nosmt

# Enable Page Table Isolation (PTI) to mitigate Meltdown
pti=on
```

Here is the same configuration in a single line suitable for copy-pasting into your bootloader configuration:

```conf
ops=panic mce=0 slab_nomerge init_on_free=1 init_on_alloc=1 page_poison=1 page_owner=1 slub_debug=FZP hardened_usercopy=1 hash_pointers=always page_alloc.shuffle=1 randomize_kstack_offset=on spec_store_bypass_disable=on intel_iommu=on amd_iommu=on iommu.passthrough=0 iommu.strict=1 vsyscall=none vdso32=0 cfi=kcfi mitigations=auto,nosmt mds=full,nosmt pti=on
```

You can find more information about these options in [Tails' kernel hardening guide](https://tails.net/contribute/design/kernel_hardening/) and [Linux Kernel Self-Protection Project Recommended Settings](https://kspp.github.io/Recommended_Settings).

Additionally, here are some additional parameters that may be too restrictive for general use, but you may consider adding them depending on your threat model and use case:

```ini
# Enable kernel lockdown mode
# Restricts loading of unsigned kernel modules and other facilities
# https://www.man7.org/linux/man-pages/man7/kernel_lockdown.7.html
lockdown=confidentiality

# Enforce kernel module signature verification
# Only allow signed kernel modules to be loaded
modules.sig_enforce=1
```

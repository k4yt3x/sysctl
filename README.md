# K4YT3X's Hardened sysctl Configuration

This repository hosts my hardened version of `sysctl.conf`. This configuration file aims to provide better security for Linux systems and improves system performance whenever possible. For example, below are some of the features this configuration file provides.

- Prevents kernel pointers from being read
- Disables Ptrace for all programs
- Disallows core dumping by SUID/GUID programs
- Disables IPv4/IPv6 routing
- Enables BBR TCP congestion control
- Enables SYN cookies to mitigate SYN flooding attacks
- Enables IP reverse path filtering for source validation
- ...

**Please review the configuration file carefully before applying it.** You are responsible for actions done to your system. If you need some guidance understanding what each of the settings is for, [sysctl-explorer](https://sysctl-explorer.net/) might come in handy. You may also consult [Linux's kernel documentation](https://www.kernel.org/doc/Documentation/sysctl/).

Please be aware that this `sysctl.conf` is **designed for 64-bit endpoint hosts that do not act as a router**. If you would like to use this configuration file on a router, please go over the configuration file and make the necessary changes (e.g., set `net.ipv4.ip_forward` to `1`).

## Configuration Deployment

Linux kernel configuration files are stored in the directory `/etc/sysctl.d`. Configurations in all files having a suffix of `.conf` will read by the `procps` (a.k.a. `systemd-sysctl`) service. Additionally, the `procps` service also loads configurations from the following directories.

- `/run/sysctl.d`
- `/usr/local/lib/sysctl.d`
- `/usr/lib/sysctl.d`
- `/lib/sysctl.d`

Files are sorted and read by their file names in lexicographic order. Variables read later will overwrite variables read earlier. For example, configurations in `20-something.conf` will be read before `99-sysctl.conf`. If a variable exists in both files, values read from `20-something.conf` will be overwritten by values read from `99-sysctl.conf`.

```properties
# in 20-something.conf
net.ipv4.ip_forward = 0

# in 99-sysctl.conf
net.ipv4.ip_forward = 1

# net.ipv4.ip_forward will be 1
```

### Method 1: Deploy Definitively

By default, on most Linux distributions, the `/etc/sysctl.d/99-sysctl.conf` file is a link to the `/etc/sysctl.conf` file. Therefore, you may write the variables into the `/etc/sysctl.conf`. However, since configuration files with a file name that starts with an alphabetical character sort later in the list than `99-sysctl.conf`, the changes you make in the `/etc/sysctl.conf` might not be the final value loaded into the kernel. To make sure that your changes are loaded into the kernel, you would have to make sure that your configuration file's name is lexicographically the last file in `/etc/sysctl.d`. The filename `z-k4yt3x.conf` will be used as an example in the code snippet below.

This deployment method is suitable for systems that do not expect to have their sysctl configurations updated from this repository anymore. Otherwise, the configuration file's content has to be updated every time a new update form this repository is installed.

```shell
# download the configuration file from GitHub using curl
curl https://raw.githubusercontent.com/k4yt3x/sysctl/master/sysctl.conf -o ~/sysctl.conf

# you may also download with wget or other methods if curl is not available
wget https://raw.githubusercontent.com/k4yt3x/sysctl/master/sysctl.conf -O ~/sysctl.conf

# move the configuration file into the sysctl configuration directory
sudo mv ~/sysctl.conf /etc/sysctl.d/z-k4yt3x.conf

# make sure the file has correct ownership and permissions
sudo chown root:root /etc/sysctl.d/z-k4yt3x.conf
sudo chmod 644 /etc/sysctl.d/z-k4yt3x.conf
```

### Method 2: Deploy as Template

Alternatively, you can use this configuration file as a template. If you name the configuration file something akin to `/etc/sysctl.d/98-k4yt3x.conf`, you may overwrite values in this configuration file by giving them a new definition the `/etc/sysctl.conf` file.

The advantage of doing this is that you would not have to change this template file's content every time it is updated in this repository. You can drop the template file in and make any modifications in `/etc/sysctl.conf`.

This method's disadvantage is that values from this template might be overwritten by values in other configurations unknowingly. For example, a `uhd-usrp2.conf` exists on my system, and overwrites the value of `net.core.rmem_max` and `net.core.wmem_max` set in previous configuration files. Packages managers can install new configurations as you install a new package or update your system. Therefore, you will have to be careful that other files do not overwrite your variables.

```shell
# download the configuration file from GitHub using curl
curl https://raw.githubusercontent.com/k4yt3x/sysctl/master/sysctl.conf -o ~/sysctl.conf

# you may also download with wget or other methods if curl is not available
wget https://raw.githubusercontent.com/k4yt3x/sysctl/master/sysctl.conf -O ~/sysctl.conf

# move the configuration file into the sysctl configuration directory
sudo mv ~/sysctl.conf /etc/sysctl.d/98-k4yt3x.conf

# make sure the file has correct ownership and permissions
sudo chown root:root /etc/sysctl.d/98-k4yt3x.conf
sudo chmod 644 /etc/sysctl.d/98-k4yt3x.conf
```

### Method 3: Custom Order (Personal Recommendation)

To ensure that the configuration files are read in an order you prefer, you may also rename the files to your preference. For example, you can install this template to `/etc/sysctl.d/y-k4yt3x.conf`, then make a symbolic link from `/etc/sysctl.d/z-sysctl.conf` to `/etc/sysctl.conf`. This ensures that the two files are more likely to be read the last.

```shell
# download the configuration file from GitHub using curl
curl https://raw.githubusercontent.com/k4yt3x/sysctl/master/sysctl.conf -o ~/sysctl.conf

# you may also download with wget or other methods if curl is not available
wget https://raw.githubusercontent.com/k4yt3x/sysctl/master/sysctl.conf -O ~/sysctl.conf

# move the configuration file into the sysctl configuration directory
sudo mv ~/sysctl.conf /etc/sysctl.d/y-k4yt3x.conf

# make sure the file has correct ownership and permissions
sudo chown root:root /etc/sysctl.d/y-k4yt3x.conf
sudo chmod 644 /etc/sysctl.d/y-k4yt3x.conf

# point z-sysctl.conf to /etc/sysctl.conf
sudo ln -s /etc/sysctl.conf /etc/sysctl.d/z-sysctl.conf
```

## Loading and Verifying the Changes

For the changes to be effective, you will have to either reboot your machine or reload the configurations using one of the following commands.

```shell
# instruct sysctl to load settings from the configuration file into the live kernel
# this command allows you to see the variables as they are being loaded
sudo sysctl --system

# alternatively, you can restart the systemd-sysctl service on a system that uses systemd
sudo systemctl restart systemd-sysctl

# procps is an alias of systemd-sysctl
# restarting either one of procps and systemd-sysctl would work
sudo systemctl restart procps
```

Afterwards, you may verify your changes by dumping all kernel variables. Replace `your.config` in the following command with the name of the variable you would like to check.

```shell
sudo sysctl -a | grep "your.config"
```

For example, the following command prints the value of `kernel.kptr_restrict`.

```shell
$ sudo sysctl -a | grep "kernel.kptr_restrict"
kernel.kptr_restrict = 2
```

## Short URL for Downloading `sysctl.conf`

For convenience, I have pointed the URL `https://k4t.io/sysctl` to the `sysctl.conf` file. You may therefore download the `sysctl.conf` file with the following command. However, be sure to check the file's integrity after downloading it if you choose to download using this method.

```shell
curl -L k4t.io/sysctl -o sysctl.conf
```

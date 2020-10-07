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

Please be careful that this `sysctl.conf` is **designed for 64-bit endpoint hosts that do not act as a router**. If you would like to use this configuration file on a router, please go over the configuration file and make necessary changes.

## Usages

1. Download the file `sysctl.conf` from the repository
1. **Review the content of the `sysctl.conf` file to make sure all settings are suitable for your system**
1. Backup your current `/etc/sysctl.conf` file (e.g., `cp /etc/sysctl.conf /etc/sysctl.conf.backup`)
1. Overwrite the old `sysctl.conf` file with the downloaded `sysctl.conf` file
1. Run command `sudo sysctl -p` or reboot the system to apply the changes

```shell
# download the configuration file from GitHub using curl
curl https://raw.githubusercontent.com/k4yt3x/sysctl/master/sysctl.conf -o ~/sysctl.conf

# you may also download with wget or other methods if curl is not available
wget https://raw.githubusercontent.com/k4yt3x/sysctl/master/sysctl.conf -O ~/sysctl.conf

# backup the original sysctl.conf
sudo cp /etc/sysctl.conf /etc/sysctl.conf.backup

# replace the old sysctl.conf with the new one
sudo mv ~/sysctl.conf /etc/sysctl.conf

# make sure the file has the correct ownership and permissions
sudo chown root:root /etc/sysctl.conf
sudo chmod 644 /etc/sysctl.conf

# instruct sysctl to load settings from the configuration file into the live kernel
sudo sysctl -p
```

For convenience, I have pointed the URL `https://akas.io/sysctl` to the `sysctl.conf` file. You may therefore download the `sysctl.conf` file with the following command. However, be sure to check the integrity of the file after downloading it if you choose to download using this method.

```shell
curl -sSL akas.io/sysctl -o sysctl.conf
```

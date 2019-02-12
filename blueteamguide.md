# CCDC - The Blue-Team Bare Minimum (Linux)

## Table of Contents
- **[Linux Basics](#linux-basics)**
	- [A note on GUIs](#a-note-on-guis)
	- [A note on scripts](#a-note-on-scripts)
	- [Sudo](#sudo)
	- [Basic navigation commands](#basic-navigation-commands)
		- [The shell](#the-shell)
		- [ls](#ls)
		- [cd](#cd)
		- [mv](#mv)
		- [cp](#cp)
		- [rm](#rm)
	- [Linux folder structure](#linux-folder-structure)
		- [/](#/)
		- [/bin](#/bin)
		- [/boot](#/boot)
		- [/etc](#/etc)
		- [/home](#/home)
		- [/tmp](#/tmp)
		- [/var](#/var)
	- [Editing text files](#editing-text-files)
	- [Man pages](#man-pages)
	- [Useful commands](#useful-commands)
		- [cat](#cat)
		- [less](#less)
		- [tail](#tail)
		- [grep](#grep)
		- [alias](#alias)
- **[Package Management](#package-management)**
	- [Searching](#searching)
	- [Updating](#updating)
	- [Installing](#installing)
	- [Removing](#removing)
	- [Listing](#listing)
	- [Tarballs](#tarballs)
- **[Network Management](#network-management)**
	- [Connecting](#connecting)
		- [GUI](#gui)
		- [nmcli](#nmcli)
		- [other](#other)
	- [ifconfig](#ifconfig)
	- [ping](#ping)
- **[Managing Services](#managing-services)**
	- [Configuring Services](#configuring-services)
	- [Monitoring Services](#monitoring-services)
	- [Restarting Services](#restarting-services)
	- [Stopping Services](#stopping-services)
- **[Basic Hardening](#basic-hardening)**
	- [Reducing attack surface](#reducing-attack-surface)
	- [Auditing](#auditing)
	- [Fail2Ban](#fail2ban)
	- [Firewall](#firewall)
	- [IDS](#ids)
	- [SSH](#ssh)
	- [sysctl](#sysctl)
	- [umask](#umask)
- **[So you got hacked](#so-you-got-hacked)**
	- [Who](#who)
	- [Kicking the hacker](#kicking-the-hacker)
		- [Finding the hacker\'s tty](#finding-the-hackers-tty)
		- [Killing the tty process](#killing-the-tty-process)
	- [Removing persistence](#removing-persistence)
		- [Changing passwords](#changing-passwords)
		- [Removing new user accounts](#removing-new-user-accounts)
		- [Removing ssh keys](#removing-ssh-keys)
	- [Finding out what happened](#finding-out-what-happened)
		- [Checking history](#checking-history)
	- [If worse comes to worst](#if-worse-comes-to-worst)
		- [Restarting](#restarting)
		- [Pulling the plug](#pulling-the-plug)

## Linux Basics

### A Note On GUIs
Most Linux servers don't come with GUIs for several reasons:
* GUIs waste CPU cycles that are better spent elsewhere.
* Any competent sysadmin can manage most services faster through the terminal than through a GUI.
* Most server management is not done via the physical box, but through SSH (i.e. Secure Shell) which does not play nicely with GUIs.
* And most importantly, GUIs pose a security risk.

This means you have to be prepared to manage a server through a command-line interface. Even if your box is supposed to come with a GUI, you don't know what kind of curveball CCDC will throw at you.

### A Note On Scripts
Do not use other people's scripts or copy commands without looking through them for several reasons:
* Something malicious could be hiding in there.
* You don't know what the script fails to cover.
* You don't know what the script does cover.
* Most importantly, you don't know what kind of curveball CCDC will throw at you.

There is a good chance CCDC will give you a machine without internet, or a machine with a non-functioning setup, or even a machine without a package manager. You need to have at least a basic idea of what do do in these scenarios. Being a script-kiddie is the easiest way to become dead in the water the instant something unexpected happens (oh, and the unexpected *will* happen). Knowing what commands to use, what they do, and why to use them is critical for success at CCDC.

### sudo
A regular Linux user cannot execute commands that change major functionality of a system. There is a special user on the system named `root` that can execute any command, but logging into a seperate user just to execute one command is inconvenient.

As a result, we have `sudo`, which allows us to execute a single command as the root user.
Before executing this command, sudo will ask you to enter *your* password, so make sure it is secure.
Making `sudo` secure is out of the scope of this guide, but any command prefaced with `sudo` can also be run without `sudo` as the root user. Look into `visudo` for more information on securing `sudo`.

### Basic navigation commands

#### The shell
When you first enter a Linux command line, you will be greeted with a screen like the following:
```
[user@hostname directory]$
```
Lets break this down:
* user      - The name of the current user.
* hostname  - The hostname of the current machine. This is mostly unused.
* directory - The name of the current directory you are in.
              If this is `~`, it means you are in your home directory.
			  Type `pwd` to **p**rint your complete **w**orking **d**irectory.
* $         - `$` means you are a normal user. If this is a `#`, it means you are the root user.

You then type commands into this shell and press enter to execute them.

#### ls
To list the files in the current directory:
```
ls
```
To list the files in a different directory:
```
ls /path/to/directory
```
Or to list the files in a subfolder of the current directory:
```
ls relative/path
```
To list *all* the files in the current directory:
```
ls -a
```
To list the the files in the current directory along with their permissions and sizes:
```
ls -l
```
To list *all* the files in the current directory along with their sizes:
```
ls -la
```
Most switches can be combined in the above manner.

#### cd
To change to a different directory:
```
cd /path/to/directory
```
Or to go to a subfolder of the current directory:
```
cd relative/path
```
To go to the parent directory:
```
cd ..
```
`..` always means "the above directory" in Linux.

#### mv
To move/rename a file/folder:
```
mv path1 path2
```

#### cp
To copy a file:
```
cp path1 path2
```
To copy a directory and its contents:
```
cp -r folder1 newpath
```
`-r` means "recursive", which means copy this directory and anything inside it.

#### rm
To remove a file:
```
rm file1
```
To remove a folder:
```
rm -rf file1
```
`-f` means "force", which means you aren't peppered with messages about "write protection" It does not mean that you can remove anything without `sudo`.

#### Linux folder structure

##### /
`/` refers to the root directory. Everything in your system descends from this path. This is in contrast to Windows where your devices are seperated like `C:/`, `D:/` etc. This means that devices are also under this path (located in `/dev`).

##### /bin
`/bin` or `/usr/bin` are where your executables are stored.

##### /boot
`/boot` is where your system boots off. This should be mounted read-only if possible because if your system cannot boot, your box is forever down. See `/etc/fstab` for more information

##### /etc
You're going to be spending a lot of time here. A vast majority of critical configuration files will be found here. As such, this directory is typically only writable for the root user. If the attacker manages to remove these configurations, it will be next to impossible to get the system running again. **KEEP BACKUPS OF YOUR SERVICE'S CONFIG**.

##### /home
`/home` stores the user information of all users on the box besides the root user, whose home directory is located at `/root`. The `~` directory refers to `/home/MY_USER`.

##### /tmp
`/tmp` is where temporary files are stored. It is also one of the only directories with universal write permission. As such, a great many exploits are based on this directory, so keeping this directory secure is important. Unfortunately, secuing `/tmp` involves some high-risk configuration which is out of the scope of this guide.

##### /var
If you are running a web server, most likely its located under `/var/www`. Other config files can be found here, so the same rules as `/etc` can be applied to this directory. Furthermore, your system's logs can be found under `/var/log/syslog`.

#### Editing text files

Being able to edit a text file without a GUI is critical, as most service management is done by editing text files. As a side note, **know your service inside and out**. This means:
* Know how to install your service (more on that in [Package Management](#package-management)
* Know how to configure it from the terminal
* Know what information it logs and where.
* Know how to fix it if something is broken.
* Most importantly, know how it can be hacked.

There are several command line editors available on most Linux systems:
* `nano`  - Beginners should use this. Editing text with this is intuitive but painful. Installed by default on most Linux systems.
* `pico`  - Pretty much the same as the above but slightly worse all around. Not installed by default on most Linux systems.
* `vim`   - Extremely harsh learning curve, but once you get good with it, you will never want to use any other editor. Learning this is almost a must if you plan to be a sysadmin or programmer as a career. See `vimtutor` for details. Installed by default on almost every linux system.
* `vi`    - Once in a blue moon, a Linux box won't have `vim` installed. In that case, `vi` has the same general functionality as `vim`, but is missing certain functionality such as visual mode. I don't think there's a single Linux box under the sun that doesn't have this available.
* `emacs` - Basically a worse `vim`. Not available out of the box on most Linux systems.

#### Man pages

You can't be expected to know every detail of every command by heart. In that case, you can type `man name_of_command` for default on what the command does and how to use it. Get used to learning about commands this way, because Google isn't available on a command-line machine.

#### Useful commands

The following commands, while not absolutely essential, will certainly make your life easier and your administration faster.

##### cat

To display the contents of a file without opening it in an editor:
```
cat file.txt
```

##### less

Sometimes the above can produce a lot of output. If you want to scroll through it instead of having it dumped on the screen as one block:
```
cat file.txt | less
```
The `|` operator means the output of the left program should be the input of the right. In more technical terms, it "pipes" the `stdout` of the first command to the `stdin` of the second.

##### tail
If you only want to see the last 50 lines:
```
cat file.txt | tail -50
```

##### grep

If you're looking for a specific keyord in an output, you can find it with:
```
cat file.txt | grep mykeyword
```
This will print out all lines that contain "mykeyword".
If you want the text it matched to be highlighted:
```
cat file.txt | grep --color mykeyword
```
If you want it to search case insensitively:
```
cat file.txt | grep -i mykeyword
```
Keep in mind the argument you pass to grep is not just a string, but a *regular expression*. This is what the *re* in g*re*p stands for. If you know how to write regular expressions, this is a powerful tool. However, the default regex dialect is a little lacking, so to use PCRE regex instead:
```
cat file.txt | grep -P ^mykeyword # matches "myheyword" at the beginning of the line
```
If you don't know regular expressions, stick to A-Z and 0-9 in your keyword. If you must search for a special character, you can match a term literally with:
```
cat file.txt | grep -F my$tring
```
To see 3 lines before and after the search:
```
cat file.txt | grep -c 3 mystring
```

##### alias

Typing is hard. `alias` makes it easier. If you keep mistyping `cd ..` as `cd..`, you can enter:
```
alias cd..=cd ..
```
This causes your shell to expand `cd..` to mean `cd ..`, which saves you keystrokes.

Or maybe you're sick of typing `grep -iP --color`:
```
alias grep=grep -iP --color
```
Keep in mind aliases only last until you log out. If you want them to persist, you will need to add the commands exactly as shown above to the end of a file called `~/.bashrc`

## Backups

You should keep backups of the following files.

#### Package Management
If you come from a Windows environment, you most likely install software by downloading an executable from some dodgy website and hoping it doesn't have malware in it.
Software on Linux is managed a little differently. Each distribution has its own "package manager", which is the way you install software. This way, you're sure of getting software that you don't have to search for and don't have to pray won't install BonziBuddy on the side. Some of these package managers are listed below:
* Debian/Ubuntu - `apt-get` or `apt`
* RHEL/CentOS   - `yum`
* Fedora        - `dnf`
* Arch/Manjaro  - `pacman`

Knowing how to use your Linux box's package manager is critical for setting it up to do basically anything.

##### Searching
If you forget the name of a package, you can always search for it with the following command:
* Debian/Ubuntu - `apt-cache search keyword` or `apt search keyword`
* RHEL/CentOS   - `yum search keyword`
* Fedora        - `dnf search keyword`

##### Updating
It is highly unlikely that CCDC will let you update your system unrestricted, but in the event you can, updating is one of the easiest and best defenses to intrusions.
* Debian/Ubuntu - `sudo apt-get update && sudo apt-get upgrade` or `sudo apt update && sudo apt upgrade`
* RHEL/CentOS   - `sudo yum update`
* Fedora        - `sudo dnf update`

##### Installing
To install a package
* Debian/Ubuntu - `sudo apt-get install keyword` or `sudo apt install keyword`
* RHEL/CentOS   - `sudo yum install keyword`
* Fedora        - `sudo dnf install keyword`

##### Removing
It is advisable to remove any unused packages (such as rsh) in order to reduce your attack surface. This can be done as follows:
* Debian/Ubuntu - `sudo apt-get remove keyword` or `sudo apt remove keyword`
* RHEL/CentOS   - `sudo yum remove keyword`
* Fedora        - `sudo dnf remove keyword`

##### Listing
* Debian/Ubuntu - `dpkg -l` or `apt list --installed`
* RHEL/CentOS   - `yum list installed`
* Fedora        - `dnf list installed`

##### Tarballs
There's a good chance you're unlucky enough to get on a machine with no internet access. In that case you're going to have to install the software manually with a tarball.
1. Go to one of the internet connected machines and get a tarball corresponding to the software you want. Tarballs typically have the extension `tar.gz`, `tar.xz`, or `.tgz`. Make sure it is a **precompiled binary** and not a source package as your server most likely doesn't have a C compiler installed.
2. Linux does not automatically mount USB devices as this is a security risk. To mount the USB:
	1. Run `ls /dev | grep sd` to see the storage devices currently mounted.
	2. Plug the USB in.
	3. Run `ls /dev | grep sd` again. A new device should have popped up. This is your USB.
	4. `sudo mount /dev/sdX /mnt`. Replace `X` with the letter of the new device.
	5. The contents of the USB will be mounted under `/mnt`. Now `cp /mnt/mytar.tgz ~` to copy the tarball to your home directory.
	6. `sudo umount /mnt` to unmount the USB.
3. `tar -xf ~/mytar.tgz` to extract the tarball.
4. There should be instructions in the extracted directory on how to install the package.

As you can tell, this is a time-consuming process, so make sure the software you're installing is critical before doing this.

### Network Management

#### Connecting

##### GUI
Normally using a GUI makes things slower, but this is one of the times a GUI can help you. *todo: pictures*
1. Open a terminal and type `nm-connection-editor`.
2. There should already be a `Wired connection` that may or may not be working, double-click on it.
3. Click the IPV4 tab.
4. The network in question most likely does not support DHCP, meaning you must give your machines static ip's. Change the dropdown from `Automatic (DHCP)` to `Manual`. Specify the below values.
	1. IP      - The IP address your machine is supposed to have.
	2. Netmask - This corresponds to your machine's "subnet". This is almost always `255.255.255.0`
	3. Gateway - This is your network's router. In our case it is the Palo Alto which has the IP `172.20.242.150`.
5. Without DHCP you will also have to specify a DNS to be able to resolve hostnames like "google.com". Put `172.20.242.10 172.20.242.150`.
	* `172.20.242.10`  - The lab DNS.
	* `172.20.242.150` - The router's DNS.
	* In the competition you will also specify the Windows 2008 DNS.
6. Use the upper-right to disconnect/reconnect.

##### nmcli
In the case you don't have a GUI, check if `nmcli` is installed with `which nmcli`. If you have it installed, follow the below steps.
1. To see your devices, use `nmcli dev status`. Look for an entry called something close to `eno1` or `eth0` that has the tag `ethernet`.
2. Run this command to create a connection: `sudo nmcli con add con-name "MYCONNECTION" ifname DEVICE" type ethernet ip4 XX.XX.XX.XX/24 gw4 172.20.242.150`
	* `nmcli con add con-name "MYCONNECTION"` - Create a new connection with the name "MYCONNECTION"
	* `ifname DEVICE` - Use the interface we found in the previous step.
	* `type ethernet` - Register this as an ethernet connection in contrast to a wireless one.
	* `ip4 XX.XX.XX.XX/24` - Set a static IPV4 address of XX.XX.XX.XX with a 255.255.255.0 subnet.
	* `gw4 172.20.242.150` - Use 172.20.242.150 as our gateway.
3. We have to set the DNS with: `sudo nmcli con mod "MYCONNECTION" ipv4-dns "172.20.242.10,172.20.242.150"`
	* `nmcli con mod "MYCONNECTION"` - Modify the connection named "MYCONNECTION"
	* `ipv4-dns "172.20.242.10 172.20.242.150` - Use the DNS at the given IPV4 addresses.
4. Finally, bring the connection up with: `sudo nmcli con up "MYCONNECTION" iface DEVICE"

##### other
TODO

#### ifconfig
To see if your computer is recognized by the router, you can type `ifconfig`. You will see something like the below
```
[n01367640@osprey ~]$ ifconfig
eth0      Link encap:Ethernet  HWaddr 00:50:56:96:54:94
          inet addr:139.62.234.6  Bcast:139.62.234.255  Mask:255.255.255.0
          inet6 addr: fe80::250:56ff:fe96:5494/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:11226578 errors:0 dropped:0 overruns:0 frame:0
          TX packets:12602011 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:2573552846 (2.3 GiB)  TX bytes:5556792101 (5.1 GiB)

eth1      Link encap:Ethernet  HWaddr 00:50:56:96:54:96
          inet addr:192.168.1.13  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::250:56ff:fe96:5496/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:3861556 errors:0 dropped:0 overruns:0 frame:0
          TX packets:3284047 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:3222531612 (3.0 GiB)  TX bytes:10607983482 (9.8 GiB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:9252 errors:0 dropped:0 overruns:0 frame:0
          TX packets:9252 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:801761 (782.9 KiB)  TX bytes:801761 (782.9 KiB)
```
The important points are below:
* `eth0`       - The name of the primary ethernet device. This is the one you want to focus on.
* `eth1`       - Fancy setups sometimes have a second ethernet device.
* `lo`         - The loopback device. This is the device that responds to `ping localhost`
* `inet addr`  - This is the most important entry. This is the IP of the current device. If this field is missing, your device failed to connect to the network.
* `Mask`       - The netmask of your device.

The rest of the output really doesn't matter for newbies at least. Remember: to search for a specific entry, you can:
```
ifconfig | grep -c 3 'inet addr'
```

#### ping
Now that your device is at least recognized by the router, you can `ping` to make sure it can communicate. `ping` is basically a way of testing a network to see who can respond to you.
1. At a minimum, you should be able to ping the gateway:
```
ping 172.20.242.150
```
If you can't ping the gateway, you either didn't connect to the correct gateway, or the gateway is down.
2. Now make sure your traffic is being routed correctly:
```
ping 172.20.242.10 # use any other machine on the network
```
If you can't ping other devices, likely the gateway does not redirect ping, the gateway is down, or the target device does not respond to ping.
3. Now try to reach the outside world.
```
ping 8.8.8.8 # google's DNS
```
If you can't ping that, either the internet is down or does not allow ping out.
4. Finally, try to see if hostnames can be resolved.
```
ping google.com
```
If this ping fails but the previous succeeds, it's likely that DNS is fukt or you specified the incorrect address.

### Managing Services

#### Configuring Services
How to configure a service is quite specific to a given service. However, it is most likely done by editing a config file in `/etc` or `/var`. Knowing how to configure your service is a must for CCDC.

#### Monitoring Services
You're going to be managing some kind of service. You better make sure it's up.
First you can check for listening services with
```
netstat -plunt
```
If there are any services listed that aren't critical for running your operation, remove them using the appropriate package manager.
If the service you're maintaining is listed, find out what's wrong with:
```
systemctl status mysqld
```
If you don't have systemctl, you can alternatively check it with
```
service mysqld status
```
If neither of those work, you'll have to look up the location of your service's logs and check them yourself. Some examples are below:
* General - `/var/log/messages` or `/var/log/syslog`
* BIND9   - `/var/log/syslog`
* MySQL   - `/var/log/mysqld.log`
* Mail    - `/var/log/maillog`

#### Restarting Services
When you edit your service's configuration file, you usually have to restart it for your changes to take effect. This can be done with:
```
systemctl restart mysqld
```
Or
```
service mysqld status
```
If neither of those work, you'll have to look up something specific for your service.

#### Stopping Services
You might have to stop a service if it is under attack and you need to get things under control. In that case:
```
systemctl stop mysqld
```
Or
```
service mysqld stop
```

### Basic Hardening

#### Auditing
You could audit by hand if you had more then 30 minutes to set things up. Fortunately, there are several tools to automatically scan your system for common vulnerabilities. One of my favorites is `lynis`, but others like `chkrootkit` exist as well.
To audit your system using `lynis`, install it through your given package manager, and then
```
sudo lynis -c
```
It will automatically scan your system for common vulnerabilities and tell you how to patch them.

#### Firewall
The Palo Alto will get hacked eventually. When that happens, you want to keep your attack surface down as much as possible, especially with the vulnerable kernel you will be using. I highly recommend the firewall `ufw`, as it is both easy to set up and installed by default on most Linux systems.

Make sure you are familliar with the port(s) your service(s) require.

First, set the default to deny both incoming and outgoing requests:
```
sudo ufw default deny outgoing
sudo ufw default deny incoming
```

Then, enable the firewall:
```
sudo ufw enable
```

If you need to disable the firewall, use:
```
sudo ufw disable
```

To allow a port incoming:
```
sudo ufw allow 22
```

To allow a port incoming only for tcp:
```
sudo ufw allow 22/tcp
```

To allow a port incoming only from a specific subnet:
```
sudo ufw allow from 172.20.242.0/24 to any port 22
```

To allow a port outgoing:
```
sudo ufw allow out 22
```

To allow a port outgoing only to a specific subnet:
```
sudo ufw allow from any to 172.20.242.0/24 port 22
```

To deny all connections from a subnet (keep in mind this probably won't be allowed at CCDC):
```
sudo ufw deny from 180.0.0.0/24
```

To check the list of rules (ufw must be active):
```
ufw status numbered
```

To then delete a rule:
```
sudo ufw delete 2  #2 is the number of the rule
```

To rate limit a port (up to 6 connections per 30 seconds):
```
sudo ufw limit 22/tcp
```

#### Fail2Ban
Fail2Ban is an IPS (i.e. Intrusion Prevention System) that detects brute forcing attempts and bans IP's that try to do so.
1. [Install](#installing) `fail2ban` using your favorite package manager and [enable the service](#restarting-services)
2. The default configuration *should* automatically start

#### IDS
An IDS, or Intrusion Detection System, tracks your system for signs of an intrusion. Note that it does not prevent intrusions nor does it help you fight of intruders. Two popular Linux based IDS's are `tiger` and `OSSEC`.

*todo: finish this section*

#### SSH
SSH (i.e. Secure Shell) is a way of logging into a system remotely. This is convenient for system administrators who do not have physical access to a machine. This is also convenient for hackers who can destroy everything if they get in. Here are some guidelines for securing it:

1. Open up the `sshd` configuration file at `/etc/ssh/sshd_config` with your favorite text editor.
2. First and foremost, deny root login by finding the commented line saying `# PermitRootLogin ues` and change it to say `# PermitRootLogin no`
*todo: ssh key*

#### sysctl
`sysctl` (not to be confused with `systemctl`) is a daemon that controls kernel parameters are runtime. These settings are essential for preventing your service from buckling under many kinds of denial of service attacks.

To prevent the common "SYN Flood" attack:
```
sudo sysctl -w net.ipv4.tcp_syncookies=1
```

To validate IP addresses (helps with spoofing):
```
sudo sysctl -w net.ipv4.conf.default.rp_filter=1
sudo sysctl -w net.ipv4.conf.all.rp_filter=1
```

To log "martian" packets (packets with an invalid destination):
```
sudo sysctl -w net.ipv4.conf.default.log_martians=1
sudo sysctl -w net.ipv4.conf.all.log_martians=1
```

To disable redirects through your box:
```
sudo sysctl -w net.ipv4.conf.all.accept_redirects=0
sudo sysctl -w net.ipv4.conf.default.accept_redirects=0
sudo sysctl -w net.ipv4.conf.all.secure_redirects=0
sudo sysctl -w net.ipv4.conf.default.secure_redirects=0
sudo sysctl -w net.ipv6.conf.all.accept_redirects=0
sudo sysctl -w net.ipv6.conf.default.accept_redirects=0
sudo sysctl -w net.ipv4.conf.all.send_redirects=0
sudo sysctl -w net.ipv4.conf.default.send_redirects=0
```

#### umask
Umask sets the permissions an application *cannot* create by default. The default value of `022` is too insecure, so change it with
```
umask 177
```

### So you got hacked

#### Who
*How do I know if my box has been breached?* Outside of your IDS, it's always nice to run the `who` command regularly, which should produce an output like:
```
[user@my-box ~]$ who
user tty1         2019-02-11 10:26
```
This shows the user logged in, the service (or tty) they're on, and the time they log in. **If there is more than one entry here, someone else is logged in to your system.**

#### Kicking the hacker
So this shows up on your `who`:
```
[user@my-box ~]$ who
user tty1         2019-02-11 10:26
user pts/3        2019-02-11 13:24 (62.62.62.62)
```
You've been hacked. Now your top priority is getting the hacker off your system.

1. See the second column next to the user name that reads `tty1` and `pts/3`? This refers to the `tty` the user is logged into. The second entry with the IP next to it is the one the hacker is using.
2. Find out the process associated with this tty with
```
[user@my-box ~]$ ps --tty pts/3
  PID TTY          TIME CMD
16210 pts/3    00:00:00 zsh
```
3. See the column all the way on the left that reads `16210` under `PID`? This is the PID of the process. Kill it below with:
```
sudo kill -9 16210
```
The `-9` specifies that we want to use the `SIGKILL` signal, which cannot be ignored.

#### Removing persistence
Any decent hacker will have put a way to log back in once they are kicked. This is how you remove that.

##### Changing passwords
The very first thing you do is change your passwords for all accounts on the system. For example, to change the password for the `root` user:
```
sudo passwd root
```
You will not be asked for the old password. You just have to put in the new password twice. If time is of the essence and you need to edit multiple passwords, you can skip the verification process with
```
echo -e "hunter2\nhunter2" | sudo passwd root
```
Keep in mind that just like when you bust a nut to your anime waifu, it's a good idea to clear your history:
```
history -c
```
This way, any future attacker won't be able to discern your ~~ waifu ~~ password.

##### Removing new user accounts
If the hacker got access to a root account, they most likely will have created a new user. You can list the users with
```
cat /etc/passwd
```
Keep in mind there will be a lot of users here. *Make sure you know how this should look by default*.
To remove a user:
```
sudo userdel redteamuser
```

##### Removing ssh keys.
A smart hacker will upload an ssh key to your box so they can continue to log in. If you do not use ssh keys, issue the following command
```
sudo rm /home/*/.ssh/authorized_keys
```
to remove any ssh keys on the computer.

If you do use ssh keys, issue the above command and then restore your old `~/.ssh/authorized_keys` file.

#### Finding out what happened

##### Checking history
To see the history of commands run on an account, just type in
```
history user
```
You may want to pipe it to less or tail to see only the commands the hacker executed.

#### If worse comes to worst

##### Restarting
You have the option of restarting your box with
```
sudo restart
```
In some CCDC competitions this will revert your box to how it was originally. Keep in mind attackers will still be trying to attack it.

##### Pulling the plug
Just like your kids, when things are fukt, you pull the plug. This will stop the bleeding and give you time to cool down and patch security holes. Revisit [Stopping Services](#stopping-services).

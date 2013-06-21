# docker-debian-repository

Recipe for building a [private debian repository](http://wiki.debian.org/SettingUpSignedAptRepositoryWithReprepro) docker container.


## Configuration

As a convention, here at [botify-labs](https://github.com/botify-labs), we set up the Docker building filesystem context
in a ``context`` subfolder mirroring the actual final container fs state, just like you would do in a [puppet]() module ``files``
folder. If you don't get it yet, take a look at the ``context`` folder of the repository ;)

#### Required context (configuration which needs you to get hands dirty a little bit)

##### /var/www/repos/apt/debian/conf/distributions

Example content:

```
Origin: Your project name
Label: Your project name
Codename: <osrelease>
Architectures: i386 amd64
Components: main
Description: Apt repository for project x
DebOverride: override.<osrelease>
DscOverride: override.<osrelease>
SignWith: <key-id>
```

*For more details, please refer to [official debian documentation](http://wiki.debian.org/SettingUpSignedAptRepositoryWithReprepro)*


##### /var/www/repos/apt/debian/conf/options

Example content:

```
verbose
basedir /var/www/repos/apt/debian
ask-passphrase
```

*For more details, please refer to [official debian documentation](http://wiki.debian.org/SettingUpSignedAptRepositoryWithReprepro)*

##### /var/www/repos/apt/debian/conf/override

```
your_package_name Priority        optional
your_package_name Section         net
```

*For more details, please refer to [official debian documentation](http://wiki.debian.org/SettingUpSignedAptRepositoryWithReprepro)*


##### /etc/nginx/sites-enabled/<your_vhost>

``debian-docker-repository`` sets up an nginx service which obviously needs a vhost to serve your packages correctly.
Here's an example of a debian repository compatible nginx vhost:

```apache
server {
  listen 80;
  server_name <hostname>;

  access_log /var/log/nginx/packages-access.log;
  error_log /var/log/nginx/packages-error.log;

  location / {
    root <absolute repos path>;
    index index.html;
  }

  location ~ /(.*)/conf {
    deny all;
  }

  location ~ /(.*)/db {
    deny all;
  }
}
```

#### Optional context (which will do the job if you're too lazy to set it up)

##### /etc/ssh/sshd_config

As a default ``docker-debian-repository`` will update the container ssh configuration to set a ``authotized_keys`` file for the ``debianrepo``
user. Feel free to use your own.
Here's the default as a reference:

```ssh
# Package generated configuration file
# See the sshd_config(5) manpage for details

# What ports, IPs and protocols we listen for
Port 22
# Use these options to restrict which interfaces/protocols sshd will bind to
#ListenAddress ::
#ListenAddress 0.0.0.0
Protocol 2
# HostKeys for protocol version 2
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_dsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
#Privilege Separation is turned on for security
UsePrivilegeSeparation yes

# Lifetime and size of ephemeral version 1 server key
KeyRegenerationInterval 3600
ServerKeyBits 768

# Logging
SyslogFacility AUTH
LogLevel INFO

# Authentication:
LoginGraceTime 120
PermitRootLogin yes
StrictModes yes

RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile	%h/.ssh/authorized_keys

# Don't read the user's ~/.rhosts and ~/.shosts files
IgnoreRhosts yes
# For this to work you will also need host keys in /etc/ssh_known_hosts
RhostsRSAAuthentication no
# similar for protocol version 2
HostbasedAuthentication no
# Uncomment if you don't trust ~/.ssh/known_hosts for RhostsRSAAuthentication
#IgnoreUserKnownHosts yes

# To enable empty passwords, change to yes (NOT RECOMMENDED)
PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication no

# Change to no to disable tunnelled clear text passwords
PasswordAuthentication no

# Kerberos options
#KerberosAuthentication no
#KerberosGetAFSToken no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes

X11Forwarding yes
X11DisplayOffset 10
PrintMotd no
PrintLastLog yes
TCPKeepAlive yes
#UseLogin no

#MaxStartups 10:30:60
#Banner /etc/issue.net

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

Subsystem sftp /usr/lib/openssh/sftp-server

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the ChallengeResponseAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via ChallengeResponseAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and ChallengeResponseAuthentication to 'no'.
UsePAM yes
```

##### /home/debianrepo/.ssh/authorized_keys

As a default, the container will host a ``debianrepo`` user which has no ssh ``authorized_keys``. If you'd wanna be able to log in to the 
container, please add the authorized public keys to it using ``cat mypublickeyfile >> ./context/home/debianrepo/.ssh/authorized_keys.


##### /etc/sudoers

As a default, the container will host a ``debianrepo`` user which is a sudoer. Feel free to update the sudoer file according to your tastes.
Here's the base file which will be deployed if you left it as is:

```ini
## Host Aliases
## Groups of machines. You may prefer to use hostnames (perhaps using 
## wildcards for entire domains) or IP addresses instead.
# Host_Alias     FILESERVERS = fs1, fs2
# Host_Alias     MAILSERVERS = smtp, smtp2

## User Aliases
## These aren't often necessary, as you can use regular groups
## (ie, from files, LDAP, NIS, etc) in this file - just use %groupname 
## rather than USERALIAS
# User_Alias ADMINS = jsmith, mikem

## Command Aliases
## These are groups of related commands...

## Networking
# Cmnd_Alias NETWORKING = /sbin/route, /sbin/ifconfig, /bin/ping, /sbin/dhclient, /usr/bin/net, /sbin/iptables, /usr/bin/rfcomm, /usr/bin/wvdial, /sbin/iwconfig, /sbin/mii-tool

## Installation and management of software
# Cmnd_Alias SOFTWARE = /bin/rpm, /usr/bin/up2date, /usr/bin/yum

## Services
# Cmnd_Alias SERVICES = /sbin/service, /sbin/chkconfig

## Updating the locate database
# Cmnd_Alias LOCATE = /usr/bin/updatedb

## Storage
# Cmnd_Alias STORAGE = /sbin/fdisk, /sbin/sfdisk, /sbin/parted, /sbin/partprobe, /bin/mount, /bin/umount

## Delegating permissions
# Cmnd_Alias DELEGATING = /usr/sbin/visudo, /bin/chown, /bin/chmod, /bin/chgrp 

## Processes
# Cmnd_Alias PROCESSES = /bin/nice, /bin/kill, /usr/bin/kill, /usr/bin/killall

## Drivers
# Cmnd_Alias DRIVERS = /sbin/modprobe

# Defaults specification
#
# Disable "ssh hostname sudo <cmd>", because it will show the password in clear. 
#         You have to run "ssh -t hostname sudo <cmd>".
#
Defaults    requiretty

Defaults    env_reset
Defaults    env_keep =  "COLORS DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR LS_COLORS"
Defaults    env_keep += "MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE"
Defaults    env_keep += "LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES"
Defaults    env_keep += "LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE"
Defaults    env_keep += "LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY"

Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin

## Allow users to run any commands anywhere 
root    ALL=(ALL)       ALL
debrepo ALL=(ALL)	NOPASSWD: ALL

## Allows members of the 'sys' group to run networking, software, 
## service management apps and more.
# %sys ALL = NETWORKING, SOFTWARE, SERVICES, STORAGE, DELEGATING, PROCESSES, LOCATE, DRIVERS

## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL

## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL

## Allows members of the users group to mount and unmount the 
## cdrom as root
# %users  ALL=/sbin/mount /mnt/cdrom, /sbin/umount /mnt/cdrom

## Allows members of the users group to shutdown this system
# %users  localhost=/sbin/shutdown -h now

## Read drop-in files from /etc/sudoers.d (the # here does not mean a comment)
#includedir /etc/sudoers.d
```

## Building

```bash
$ git clone git@github.com:botify-labs/docker-debian-repository
$ cd docker-debian-repository
$ docker build -t debian-repository debian-repository
```

And, considering you've set every required context files, you should be done.


## Running

``docker-debian-repository`` exposes the nginx service through port 80 and ssh through port 22. You'll probably want to route your host
port to localshop container using the ``-p`` option like the following:

```bash
$ docker run -d -p LOCAL_PORT:CONTAINER_PORT docker-repository
```

For example, you could route port 4444 to 80 and proxy the ports using nginx in order not to conflict with other potential port 80 hosts
on your machine: 

```bash
$ docker run -d -p 4444:80 debian-repository
```

Or/And you could route port 5555 to 22 in order to ssh directly to the container from the outside using port 5555 (don't forget to update the ``authorized_keys`` file to enable ssh over the container):

```bash
docker run -d -p 5555:22 debian-repository
```


## License

Copyright (c) 2013, Theo Crevon. All rights reserved.


Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.


## Contribute

Localshop container builder is far from perfect, and could be easily enhanced or fixed, don't restrain yourself
just follow the path:

1. Check for open issues or open a fresh issue to start a discussion around a feature idea or a bug.
2. Fork the repository on Github to start making your changes to the master branch.
3. Send a pull request and bug the maintainer until it gets merged and published.
4. Make sure to add yourself to AUTHORS.


*Please note that these instructions were lazily copied from [Kenneth reitz requests](https://github.com/kennethreitz/requests)*

# ubuntu-autoinstall
I want to learn how to perform a automatic installation of an Ubuntu Server.

There is a slight difference to how it is done with a Debian system The differences are laid out [here](https://ubuntu.com/server/docs/install/autoinstall) in the quickstart guide.

From what I understand there are three options to perform an autoinstall with an automatic configuration:
* Modify the ISO file
* Serve the configuration over network
* By using another volume

For my use case I will concentrate on the network way.

# Getting the information from the network (HTTP/HTTPS)
In order to perform a fully automatic installation the following options has to be added to the kernel prompt:
* autoinstall
* (in case of network installation)  ds=nocloud-net;s=http://<ip>:<port>/

More information about the NoCloud Provider of the Cloud-Init stacak can be found [here](https://cloudinit.readthedocs.io/en/latest/topics/datasources/nocloud.html)

## Files needed on the HTTP server
These files are required to generate a valid ISO file, according to the Cloud-Init documentation.
* user-data
* meta-data

But the Ubuntu quickstart guide only mentions:
* user-data

So, I guess only the user-data is required in this case.

# Configuration
[Reference](https://ubuntu.com/server/docs/install/autoinstall-reference)
[Example](https://ubuntu.com/server/docs/install/autoinstall)

The following example is in the Example. Just in case the documentation will be changed
```
version: 1
reporting:
    hook:
        type: webhook
        endpoint: http://example.com/endpoint/path
early-commands:
    - ping -c1 198.162.1.1
locale: en_US
keyboard:
    layout: gb
    variant: dvorak
network:
    network:
        version: 2
        ethernets:
            enp0s25:
               dhcp4: yes
            enp3s0: {}
            enp4s0: {}
        bonds:
            bond0:
                dhcp4: yes
                interfaces:
                    - enp3s0
                    - enp4s0
                parameters:
                    mode: active-backup
                    primary: enp3s0
proxy: http://squid.internal:3128/
apt:
    primary:
        - arches: [default]
          uri: http://repo.internal/
    sources:
        my-ppa.list:
            source: "deb http://ppa.launchpad.net/curtin-dev/test-archive/ubuntu $RELEASE main"
            keyid: B59D 5F15 97A5 04B7 E230  6DCA 0620 BBCF 0368 3F77
storage:
    layout:
        name: lvm
identity:
    hostname: hostname
    username: username
    password: $crypted_pass
ssh:
    install-server: yes
    authorized-keys:
      - $key
    allow-pw: no
snaps:
    - name: go
      channel: 1.14/stable
      classic: true
debconf-selections: |
    bind9      bind9/run-resolvconf    boolean false
packages:
    - libreoffice
    - dns-server^
user-data:
    disable_root: false
late-commands:
    - sed -ie 's/GRUB_TIMEOUT=.*/GRUB_TIMEOUT=30/' /target/etc/default/grub
error-commands:
    - tar c /var/log/installer | nc 192.168.0.1 1000
```
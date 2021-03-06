---
Title:  Install & Configure Suricata
Description: Install and Maintain Suricata on Debian
Category: DPI 
---

This page is for setting up and configure Suricata network threat detection (version 6.0.1) engine on Debian 10.

## Requirements

* Clean Debian 10 installation with all updates installed;
* Have a **STATIC** IPv4 and IPv6 address configured on your external 
  interface;
* Network equipment/VM platform allows access to the very least `tcp/80`, 
  `tcp/443`, for basic functionality, and fetching and installing dependency packages.



## Suricata Installation On Debian 10

Suricata is a free-to-use and open-source network threat detection engine. In this documentation, there are two ways of installing Suricata, From source or the binary package. For people familiar with compiling their own software, the Source method is recommended.

### Install Suricata from Binary package 

With a simple install command you can install Suricata from the binary package in Debian 10:

	$ sudo apt-get install suricata


In the “stable” version of Debian, Suricata is usually not available in the latest version. A more recent version is often
available from Debian back-ports, if it can be built there.

To use back-ports, the back-ports repository for the current stable distribution needs to be added to the system-wide
sources list. For Debian 10 (buster), for instance, run the following as root:

	$ echo "deb http://http.debian.net/debian buster-backports main" > \
	/etc/apt/sources.list.d/backports.list
	$ apt-get update
	$ apt-get install suricata -t buster-backports


More information about installing Suricata in other distributions and various configurations can be found in this [PDF guide.](https://buildmedia.readthedocs.org/media/pdf/suricata/latest/suricata.pdf).






### Install Suricata from source

Before installing Suricata from the source, let's begin with updating the system and upgrading all packages, then reboot:

	$ sudo apt update
	$ sudo apt upgrade -y
	$ reboot 

We’ll install Suricata on Debian 10 from the source distribution files, which gives more control on Suricata installation, Before installing Suricata, several dependency packages and pre-requisite files needed to be installed, they can be installed using the following commands:

	$ apt-get install make autoconf automake libtool

	$ apt-get -y install libpcre3 libpcre3-dbg libpcre3-dev build-essential libpcap-dev \
      libnet1-dev libyaml-0-2 libyaml-dev pkg-config zlib1g zlib1g-dev liblz4-dev \
      libcap-ng-dev libcap-ng0 libmagic-dev libjansson-dev libnspr4-dev \
      libnss3-dev libgeoip-dev liblua5.1-dev libhiredis-dev libevent-dev \
      python-yaml python3-distutils python3-pip 

 Then, install this python module using pip3:

 	$pip3 install PyYAML

Remove any existing rustc package if any, and install the latest rustc package using this commands:

	$ sudo apt remove --purge rustc
	$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y

Then, Source the file:

	$ source $HOME/.cargo/env

Install Suricata from source file:

	$ suricata_select_version="6.0.1"
	$ rm -fv "suricata-${suricata_select_version}.tar.gz"
	$ wget "https://www.openinfosecfoundation.org/download/suricata-${suricata_select_version}.tar.gz"
	$ tar xzvf "suricata-${suricata_select_version}.tar.gz"
	$ cd "suricata-${suricata_select_version}/"
	$ ./configure --libdir=/usr/lib64 --prefix=/usr/local --sysconfdir=/etc --localstatedir=/var --datarootdir=/usr/local/share --enable-lua --enable-rust
	$ sudo make install-full

You can now start Suricata by running as root something like:

	$ /usr/local/bin/suricata -c /etc/suricata/suricata.yaml -i eth0

If a library like libhtp.so is not found, you can run Suricata with:
 
	$ LD_LIBRARY_PATH=/usr/local/lib /usr/local/bin/suricata -c /etc/suricata/suricata.yaml -i eth0

	


Then create the Suricata service manually:

	nano /etc/systemd/system/suricata.service

	[Unit]
	Description=Suricata IDS/IDP Service
	Wants=network.target syslog.target
	After=network.target syslog.target
	Documentation=man:suricata(8) man:suricatasc(8)
	Documentation=https://redmine.openinfosecfoundation.org/projects/suricata/wiki

	[Service]
	Type=forking
	Environment=LD_PREDLOAD=/usr/lib/x86_64-linux-gnu/libtcmalloc_minimal.so.4
	# Debug level ---> -v: INFO | -vv: INFO+PERF | -vvv: INFO+PERF+CONFIG | -vvvv: INFO+PERF+CONFIG+DEBUG
	# D - means in daemon | -c read config | --pidfile <file> write pidfile on a file
	ExecStart=suricata --af-packet -vvv -D -c /etc/suricata/suricata.yaml --pidfile /var/run/suricata.pid
	ExecStartPre=rm -f /var/run/suricata.pid
	ExecStop=kill $MAINPID && rm -f /var/run/suricata.pid
	ExecReload=kill -9 $MAINPID

	[Install]
	WantedBy=multi-user.target


Then start Suricata and check the status:

	$ systemctl start suricata 

	$ systemctl status suricata

That should validate Suricata is active and running. 

You could test also with this command:
	
	$ sudo suricata -T




## Suricata Configuration On Debian 10

a simple configuration for Suricata is to specify the network interface in `/etc/suricata/suricata.yaml` :
	
	af-packet:
    interface: enp1s0

it depends on what is your network interface that you want Suricata to monitor, it could be  `eth0`, you could check what network platform the device has using  ` ifconfig `.

Also, specify the network range you want to monitor in `/etc/suricata/suricata.yaml` in `address-groups`:	

	  address-groups:
    #HOME_NET: "[192.168.1.0/24]"
    #HOME_NET: "[192.168.0.0/16]"
    #HOME_NET: "[10.0.0.0/8]"
    #HOME_NET: "[172.16.0.0/12]"
    #HOME_NET: "any"


### Suricata Rules 

Suricata rules can be found in  `/var/lib/suricata/rules/suricata.rules`, by default. To download the Emerging Threats Open ruleset or update ruleset, it is enough to simply run:

		$ sudo suricata-update

It is recommended to update your rules frequently.

You could use see other rule-sets available: by fetching the master index from the OISF hosts, then list available rule-sets:

	$ sudo suricata-update update-sources
	$ sudo suricata-update list-sources

It will give a list of rule-sets as a result. Each of the rule-sets has a name that has a ‘vendor’ prefix, followed by a set name. For example, secure workers security-malware rule-set is called ‘scwx/malware’. To enable ‘scwx/malware’, enter:

	$ sudo suricata-update enable-source scwx/malware
	$ sudo suricata-update

Then restart the Suricata service and the rule-set is loaded. 

You also can run testing with:
	
	$ sudo suricata -T

You should receive a notice "Configuration provided was successfully loaded. Exiting."

#### Controlling Suricata rules

By default all Suricata rules are merged into a single file `/var/lib/suricata/rules/suricata.rules`, To enable rules that are disabled by default, use `/etc/suricata/enable.conf` :

		2019401                   # enable signature with this sid
		group:emerging-icmp.rules # enable this rulefile
		re:trojan                 # enable all rules with this string

It is similar to disable any rule, use  `/etc/suricata/disable.conf` :

		2019401                   # disable signature with this sid
		group:emerging-info.rules # disable this rulefile
		re:heartbleed             # disable all rules with this string

Also, the same for `drop.conf` and `modify.conf` files. after configuring or modifying the rules, update with the command:

	$ sudo suricata-update

Then, restart Suricata:

	$ systemctl restart suricata
---
title:  Install & Configure Suricata
description: Install and Maintain Suricata on Debian
category: DPI installation
---


## Suricata Installation On Debian 10

Suricata is a free to use and open source network threat detection engine. Before installing Suricata, lets begin woth updating the system and upgrading all packages,then reboot:

	$sudo apt update
	$sudo apt upgrade -y
	$reboot 

We’ll install Suricata on Debian 10 from the source distribution files, it gives more controll on Suricata installation, Before installing Suricata, a number of dependency packages and pre-requisite files needed to be installed, they can be installed using the following commands:

	$apt-get install make autoconf automake libtool

	$apt-get -y install libpcre3 libpcre3-dbg libpcre3-dev build-essential libpcap-dev \
      libnet1-dev libyaml-0-2 libyaml-dev pkg-config zlib1g zlib1g-dev liblz4-dev \
      libcap-ng-dev libcap-ng0 libmagic-dev libjansson-dev libnspr4-dev \
      libnss3-dev libgeoip-dev liblua5.1-dev libhiredis-dev libevent-dev \
      python-yaml python3-distutils python3-pip 

 Then, install this python module using pip3:

 	$pip3 install PyYAML

Remove any existing rustc packag if any, and install the latest rustc package using this commands:

	$sudo apt remove --purge rustc
	$curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y

Then, Source the file:

	$source $HOME/.cargo/env

Install Suricata from source file:

	$suricata_select_version="6.0.1"
	$rm -fv "suricata-${suricata_select_version}.tar.gz"
	$wget "https://www.openinfosecfoundation.org/download/suricata-${suricata_select_version}.tar.gz"
	$tar xzvf "suricata-${suricata_select_version}.tar.gz"
	$cd "suricata-${suricata_select_version}/"
	$./configure --libdir=/usr/lib64 --prefix=/usr/local --sysconfdir=/etc --localstatedir=/var --datarootdir=/usr/local/share --enable-lua --enable-rust
	$sudo make install-full

Then create the suricata service manually:

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


Then start Surricata and check the status:

	$systemctl start suricata 

	$systemctl status suricata

That should validate Suricata is active and running. 



## Suricata Configuration On Debian 10
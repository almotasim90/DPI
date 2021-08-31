---
Title:  Send Suricata alerts through SMTP
Description: sending Suricata alerts as attachments from Debian to SURF mails.
Category: DPI 
---

This page is for sending Suricata alerts emails from Linux distributions to ADMIN users (SURF mail) or any other email providers.

## Requirements

* Clean Debian 10 installation with all updates installed;
* Have a **STATIC** IPv4 and IPv6 address configured on your external 
  interface;



## Install MSMTP into Debian 10

MSMTP is an SMTP client that can be used to send mails from MUAs (mail user agents) like Mutt or Mpack. It forwards mails to an SMTP server such as a free mail provider, which takes care of the final delivery. Using profiles, it can be easily configured to use different SMTP servers with different configurations, which makes it ideal for mobile clients.

First update and upgrade the server system:

	$ sudo apt-get update
	$ sudo apt-get upgrade

Then, you can easily install MSMTP with this command:

	$ sudo apt-get -y install msmtp

You need to have a file that contains Certificate Authority (CA) certificates so that we can connect using SSL / TLS to the email server. You can check whether your server already has a ca-certificates package installed or not using the command below: 

	$ dpkg -l | grep ca-certificates

If a ca-certificates package is already installed, you will get output similar to the following output:
`ii ca-certificates 20141019ubuntu0.14.04.1 all Common CA certificates` 

**NOTE** If you get blank output, it means that the ca-certificates package is not installed, and you need to install it. You can install this package by running this command:

	$ sudo apt-get -y install ca-certificates



## MSMTP Configuration 

Create an MSMTP configuration on `/etc/msmtprc` with the content below. You will have to enter your Gmail username and password on this file. Also, enter your SURF mail information. So, the `/etc/msmtprc` will look like the following:

	# Set default values for all following accounts.
	defaults
	auth off
	tls on
	tls_trust_file /etc/ssl/certs/ca-certificates.crt
	logfile ~/.msmtp.log 

	# Gmail
	account gmail
	auth on
	host smtp.gmail.com
	port 587
	from <Your email address>
	user <Your User or Email>
	password 

	# SURF
	account surf
	host outgoing.mf.surf.net
	port 25
	from 
	user 
	password

	# Set a default account
	account default : surf

**NOTE** If your account supports two-factor authentication, then you need extra steps:
	Create a custom app in your Gmail security settings.
		1. Log in into Gmail with your account
		2. Navigate to https://security.google.com/settings/security/apppasswords
		3. In 'select app' choose 'custom', give it an arbitrary name and press generate
		4. It will give you 16 chars token.
	Use the token as a password in combination with your full Gmail account and two-factor authentication will not be required. For other account providers, you can set authentication off. 

### Sending email using MSMTP 

You can use the following command to send a simple email using MSMPT:

		$ echo "Hello this is sending an email using msmtp" | msmtp <receiver email>

However, If you want to send an email with attachments, then install Mutt. 



## Mutt Installation 

By using the following installing command, you can install Mutt into Linux distribution:

	$ sudo apt-get -y install mutt

### Configuring Mutt
You will create a local configuration for Mutt on `~/.muttrc`. This file might not exist yet, you can create this file with the content below:

	$ set sendmail="/usr/bin/msmtp"
	  set use_from=yes
	  set realname="Your name"
	  set from= <You email address>
	  set envelope_from=yes

Now you can send an email with attachments using this command:

		$ echo "Message Body Here" | mutt -s "Subject Here" <receiver email> -a ~/test.txt

**NOTE** You can send an email to multiple receivers, by providing a comma between the emails. for example:
		
		$ echo "Message Body Here" | mutt -s "Subject Here" email@gmail.com, another_email@gmail.com -a ~/test.txt


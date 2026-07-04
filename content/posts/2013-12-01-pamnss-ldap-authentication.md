---
title: PAM & NSS LDAP Authentication
date: 2013-12-01T23:31:21
draft: true
tags:
  - linux
  - tech
---

The amount of servers and services I administrate and manage is rising slowly but steadily. I started with one little VPS and now I have 8 Virtual Machines — the three test environments not included. Clearly the need for a centralized authentication platform was given.

Since I am using Ubuntu (12.04) for my servers, this post will describe how to install and configure LDAP client authentication on Ubuntu systems. This ‘guide’ is

## 389 – LDAP server

First thing to do is to install the 389 LDAP Directory Server from the Ubuntu package repositories:

`apt-get install 389-admin 389-admin-console 389-console 389-ds 389-ds-base 389-ds-base-libs 389-ds-console`

To complete the 389 Directory Server installation and configuration, just run:

`/usr/sbin/setup-ds`

Answer the questions asked during the setup. I would recommend to take some time to read and understand the setup questions properly.

After the 389 DS is properly installed, configured and running, the LDAP user and group needs to be created and imported to the DS. To achieve this, generate the following base.ldif file with your favourite editor:
    
    
    ## An example entry to add a user to LDAP
    ###
    dn: cn=aaronson,ou=people,dc=mydomain,dc=com
    objectClass: inetOrgPerson
    objectClass: organizationalPerson
    objectClass: person
    objectClass: posixAccount
    cn: aaronson
    sn: Aronson
    givenName: Aaron
    uid: aaronson
    userPassword: my-secrect-password
    mail: me@mydomain.com
    telephonenumber: 1234567890
    l: Town
    postalcode: 12345
    uidNumber: 5001
    gidNumber: 5001
    homeDirectory: /export/home/aaronson
    loginShell: /bin/bash
    ###
    ## An example entry to add a group to LDAP (same name as the user)
    ###
    dn: cn=aaronson,ou=groups,dc=mydomain,dc=com
    objectclass: top
    objectclass: posixGroup
    cn: aaronson
    gidnumber: 5001
    memberuid: aaronson
    

Then use ldapadd to add the user to the 389 LDAP server:

`ldapadd -x -f base.ldif -D "cn=Directory Manager" -w mysecretpassword`

## NFS Server

After importing the LDIF file to the DS server, we need to prepare the NFS server so that it shares the user’s home directory and can be (auto)mounted during login.

Edit the ‘/etc/exports’ file to export the share where the home directories will be created on:
    
    
    /export         192.168.0.0/24(rw,fsid=0,no_subtree_check,sync)
    /export/home    192.168.0.0/24(rw,no_subtree_check,sync)
    

After restart the NFS server, you should be good to go. Test your configuration by manually mounting the share on your servers. If it works, you can go ahead and edit the pam files. If not, consult the NFS manpage (especially when you have permissions issues, this can be tricky sometimes).

Also edit the file ‘/etc/pam.d/common-session’ and add the following entries at the end of the file:
    
    
    # end of pam-auth-update config
    session    required    pam_mkhomedir.so    skel=/etc/skel    umask=0022
    

This makes sure that the user’s home directory is created during login if it does not exist. This will only happen when you login to the file server for the first time. This is some lazy workaround for me, as I will explain at the end of this post.

## Client LDAP configuration

Now the clients need to be configuration. Install the following packages on your designated LDAP client:

`apt-get install auth-client-config ldap-auth-client ldap-auth-config libnss-ldap libpam-ldap`

You will be asked a variety of questions similar to the those asked when you were installing the server components:
    
    
    LDAP server Uniform Resource Identifier: ldap://**LDAP-server-IP-address**
    
    Change the initial string from "ldapi:///" to "ldap://" before inputing your server's information
    
    Distinguished name of the search base:
    Our example was "dc=mydomain,dc=com"
    LDAP version to use: 3
    
    Make local root Database admin: Yes
    
    Does the LDAP database require login? No
    
    LDAP account for root: uid=admin,ou=Administrators,ou=TopologyManagement,o=netscaperoot
    LDAP root account password: secret-ldap-admin-password
    

vim /etc/ldap/ldap.conf
    
    
    BASE        ou=people,dc=mydomain,dc=com
    URI         ldap://ldapserver.mydomain.com:389
    #SIZELIMIT  12
    #TIMELIMIT  15
    DEREF       never
    # TLS certificates (needed for GnuTLS)
    TLS_CACERT  /etc/ssl/certs/ca-certificates.crt
    

vim /etc/nsswitch.conf
    
    
    passwd: ldap compat
    group:  ldap compat
    shadow: ldap compat
    

First, edit the /etc/nsswitch.conf file. This will allow us to specify that the LDAP credentials should be modified when users issue authentication change commands.

sudo nano /etc/nsswitch.conf  
The three lines we are interested in are the “passwd”, “group”, and “shadow” definitions. Modify them to look like this:
    
    
    passwd:         ldap compat
    group:          ldap compat
    shadow:         ldap compat
    

### Client Automount configuration

Install autofs packages:

`apt-get install autofs-ldap autofs`

Add the following lines to the autofs configuration files. First of all the master file:

`vim /etc/auto.master`
    
    
    # Sample auto.master file
    # This is an automounter map and it has the following format
    # key [ -mount-options-separated-by-comma ] location
    # For details of the format look at autofs(5).
    #
    #/misc    /etc/auto.misc
    #
    # NOTE: mounts done from a hosts map will be mounted with the
    #    "nosuid" and "nodev" options unless the "suid" and "dev"
    #    options are explicitly given.
    #
    #/net    -hosts
    #
    # Include central master map if it can be found using
    # nsswitch sources.
    #
    # Note that if there are entries for /net or /misc (as
    # above) in the included master map any keys that are the
    # same will not be seen as the first read key seen takes
    # precedence.
    #
    +auto.master
    
    /export/home    /etc/auto.home
    

Then edit the autofs config file for the home directories ‘/etc/auto.home’:
    
    
    *    -fstype=nfs,soft,intr,rsize=8192,wsize=8192,nosuid,tcp  ldapserver.mydomain.com:/export/home/&
    

The last step needed is the creation of the users home directory on the NFS share so that it can be (auto-)mounted every time a LDAP user logs in. You can either write yourself scripts or a web UI for this or do the first login on the file server (remember the NFS server configuration above).

After all these configuration steps you should be able to login to every host with your LDAP credentials.

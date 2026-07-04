---
title: Use Touch ID with sudoers
date: 2024-01-28T02:38:37
tags: [macos, security, sudo]
languages: [en]
feature_image: https://media.jason.re/2024/01/28/sudo.jpg
---

Thanks to a discussion in IRC about Touch ID, I remembered that I always wanted to enable it for `sudo`. Fortunately, that does not take long:

```shell
$ sudo cat /etc/pam.d/sudo
# sudo: auth account password session
auth       sufficient     pam_smartcard.so
auth       required       pam_opendirectory.so
account    required       pam_permit.so
password   required       pam_deny.so
session    required       pam_permit.so
$ sudo gsed -i '2 i\auth       sufficient     pam_tid.so' /etc/pam.d/sudo
$ sudo cat /etc/pam.d/sudo
# sudo: auth account password session
auth       sufficient     pam_tid.so
auth       sufficient     pam_smartcard.so
auth       required       pam_opendirectory.so
account    required       pam_permit.so
password   required       pam_deny.so
session    required       pam_permit.so
```

Works like a charm. Now you can use Touch ID with `sudo` in the console. If it is that easy to enable, using it for private SSH keys should not require much more effort:

```shell
# configure your ssh client
$ cat ~/.ssh/config
AddKeysToAgent yes
UseKeychain yes

Host *
  ServerAliveInterval 30
  IdentityFile ~/.ssh/id_rsa

# add the key to the apple keychain
$ ssh-add --apple-use-keychain .ssh/id_rsa
Identity added: .ssh/id_rsa (jason@orthanc.local)
```

Now that works too! Don't forget to load your key with `ssh-add --apple-load-keychain .ssh/id_rsa` when you want to use it.

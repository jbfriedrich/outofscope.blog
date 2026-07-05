---
title: "Ansible: Misleading dopy errors"
date: 2017-05-14T19:33:25
tags:
  - linux
  - automation
languages:
  - en
---

[Ansible](https://www.ansible.com/) works pretty well with [DigitalOcean](https://www.digitalocean.com/), and where it is not, it is mostly the fault of [dopy](https://pypi.python.org/pypi/dopy), not DigitalOcean’s or Ansible’s. I am using macOS and out of convenience I installed Ansible via [Homebrew](https://brew.sh/). When trying certain actions, for example destroying a Droplet, I got this error message:

`dopy >= 0.2.3 required for this module`

This was weird, as I was sure I had installed dopy via pip (and pip via Homebrew). After double-checking and some digging I found the issue to be with the Ansible installation. To fix the issue at hand, one can [either](https://groups.google.com/forum/?hl=ru#!topic/ansible-project/gjQM-rArtTg) add

`localhost ansible_connection=local ansible_python_interpreter=python`

to the hosts file, [or](https://github.com/ansible/ansible-modules-core/issues/360) run

`/usr/local/Cellar/ansible/2.3.0.0_2/libexec/bin/pip install dopy==0.3.7a`.

---
title: Let's Encrypt Wildcard Certificates with Docker
date: 2019-07-07T10:58:53
feature_image: https://images.unsplash.com/photo-1550751827-4bd374c3f58b?ixlib=rb-1.2.1&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=1080&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ
tags:
  - tech
  - security
  - virtualization
  - container
languages:
  - en
---

In the past I used a self-built Docker container that was running easy-rsa with a customized openssl.cnf file. This got very annoying, very quickly, as I needed to import my private CA to all systems I wanted to use it on.

A couple of weeks back I decided to use one of my domains, _frdr.ch_ , to use for my internal systems. This means, only the main domain _frdr.ch_ will be exposed to the outside word, all other entries will only be available on the internal DNS server that overrides the required local entries.

Since the domain itself is public, and [Let's Encrypt](https://letsencrypt.org/) offers [Wildcard Certificates](https://community.letsencrypt.org/t/acme-v2-and-wildcard-certificate-support-is-live/55579) for a while now, I decided to go that route and finally ditch my [easy-rsa](https://github.com/OpenVPN/easy-rsa) solution. This also would allow me to automate the certificate creation/renewal with [Certbot](https://certbot.eff.org/) and keep the operational overhead to a minimum.

Before I roll my own solution, I wanted to see if someone already came up with a good solution. And I was lucky. I found [Adrien Ferrand's](https://github.com/adferrand) [solution](https://github.com/adferrand/docker-letsencrypt-dns) that is using Docker, Certbot and [Lexicon](https://github.com/AnalogJ/lexicon).

## Creating the required configuration files

For easier backups, my Docker servers use bind volumes whenever possible. This makes deploying my configuration files rather easy.

```shell
$ mkdir /opt/docker-sync/le-dns
$ cd /opt/docker-sync/le-dns
$ vim domains.conf
  *.frdr.ch frdr.ch
$ vim lexicon.yml
  # Content of /etc/letsencrypt/lexicon.yml
  delegated: frdr.ch
  inwx:
    auth_username: username
    auth_password: super_secret_password
```

## Building your own Docker Image (optional)

To be as independent as possible, I like to fork projects and build my own Docker Images. This is not required though, it can work out-of-the-box with Adrien's repository.

```shell
$ docker build --no-cache --force-rm -t <redacted>.dkr.ecr.eu-central-1.amazonaws.com/le-dns:latest -t <redacted>.dkr.ecr.eu-central-1.amazonaws.com/le-dns:1.3 .
Sending build context to Docker daemon  35.84kB
Step 1/18 : FROM python:alpine3.9
 ---> fe3ef29c73f3
Step 2/18 : LABEL maintainer="Jason Friedrich <jason@friedrich>"
 ---> Running in 1839c18d6113
Removing intermediate container 1839c18d6113
 ---> d7183496d232
Step 3/18 : ENV PATH /scripts:$PATH
 ---> Running in 87820e865295
Removing intermediate container 87820e865295
 ---> 7ee5ecb6c396
Step 4/18 : ENV LEXICON_VERSION 3.2.7
 ---> Running in ac8fbc70df37
Removing intermediate container ac8fbc70df37
 ---> 4d4a2d4cabc9
Step 5/18 : ENV CERTBOT_VERSION 0.35.1
 ---> Running in 0f940bccc60e
Removing intermediate container 0f940bccc60e
 ---> 742ee43407f7
Step 6/18 : RUN apk --no-cache --update add rsyslog git libffi libxml2 libxslt libstdc++ openssl docker ethtool tzdata bash  && apk --no-cache --update --virtual build-dependencies add libffi-dev libxml2-dev libxslt-dev openssl-dev build-base linux-headers  && pip install --no-cache-dir "certbot==$CERTBOT_VERSION"  && pip install --no-cache-dir "dns-lexicon[full]==$LEXICON_VERSION"  && pip install --no-cache-dir circus  && mkdir -p /var/lib/letsencrypt/hooks  && mkdir -p /etc/circus.d  && apk del build-dependencies
 ---> Running in 303e1b43f573
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/community/x86_64/APKINDEX.tar.gz
(1/28) Installing bash (4.4.19-r1)
Executing bash-4.4.19-r1.post-install
(2/28) Installing libseccomp (2.3.3-r1)
(3/28) Installing runc (1.0.0_rc8-r0)
(4/28) Installing containerd (1.2.7-r0)
[...]
Step 16/18 : VOLUME ["/etc/letsencrypt"]
 ---> Running in 3dda9d00bc18
Removing intermediate container 3dda9d00bc18
 ---> 0cec67ab7143
Step 17/18 : COPY files/lexicon.yml /etc/letsencrypt/lexicon.yml
 ---> 8a17a1cf7880
Step 18/18 : CMD ["/scripts/run.sh"]
 ---> Running in aaebb4dfd276
Removing intermediate container aaebb4dfd276
 ---> d7f84f270f34
Successfully built d7f84f270f34
Successfully tagged <redacted>.dkr.ecr.eu-central-1.amazonaws.com/le-dns:latest
Successfully tagged <redacted>.dkr.ecr.eu-central-1.amazonaws.com/le-dns:1.3
```

## Run your Docker container

```shell
$ docker run --rm --name le-dns --volume /opt/docker-sync/le-dns:/etc/letsencrypt <redacted>.dkr.ecr.eu-central-1.amazonaws.com/le-dns:latest
2019-07-07 09:31:32 circus[1] [INFO] Starting master on pid 1
2019-07-07 09:31:32 circus[1] [INFO] Arbiter now waiting for commands
2019-07-07 09:31:32 circus[1] [INFO] crond started
2019-07-07 09:31:32 circus[1] [INFO] watch-domains started
2019-07-07 09:31:32 [19] | #### Registering Let's Encrypt account if needed ####
2019-07-07 09:31:33 [19] | Saving debug log to /etc/letsencrypt/logs/letsencrypt.log
2019-07-07 09:31:33 [19] | There is an existing account; registration of a duplicate account with this command is currently unsupported.
2019-07-07 09:31:33 [19] | #### Clean autorestart/autocmd jobs
2019-07-07 09:31:33 [19] | #### Creating missing certificates if needed (~1min for each) ####
2019-07-07 09:31:33 [19] | >>> Creating a certificate for domain(s): -d *.frdr.ch -d frdr.ch
2019-07-07 09:31:34 [19] | Saving debug log to /etc/letsencrypt/logs/letsencrypt.log
2019-07-07 09:31:34 [19] | Plugins selected: Authenticator manual, Installer None
2019-07-07 09:31:34 [19] | Obtaining a new certificate
2019-07-07 09:31:35 [19] | Performing the following challenges:
2019-07-07 09:31:35 [19] | dns-01 challenge for frdr.ch
2019-07-07 09:31:35 [19] | dns-01 challenge for frdr.ch
2019-07-07 09:31:35 [19] | Running manual-auth-hook command: /var/lib/letsencrypt/hooks/authenticator.sh
2019-07-07 09:32:08 [19] | Output from manual-auth-hook command authenticator.sh:
2019-07-07 09:32:08 [19] | RESULT
2019-07-07 09:32:08 [19] | ------
2019-07-07 09:32:08 [19] | True
2019-07-07 09:32:41 [19] | Waiting for verification...
2019-07-07 09:32:42 [19] | Cleaning up challenges
2019-07-07 09:32:42 [19] | Running manual-cleanup-hook command: /var/lib/letsencrypt/hooks/cleanup.sh
2019-07-07 09:32:45 [19] | Output from manual-cleanup-hook command cleanup.sh:
2019-07-07 09:32:45 [19] | RESULT
2019-07-07 09:32:45 [19] | ------
2019-07-07 09:32:45 [19] | True
2019-07-07 09:32:50 [19] | Non-standard path(s), might not work with crontab installed by your operating system package manager
2019-07-07 09:32:50 [19] | Running deploy-hook command: deploy-hook.sh
2019-07-07 09:32:50 [19] | IMPORTANT NOTES:
2019-07-07 09:32:50 [19] |  - Congratulations! Your certificate and chain have been saved at:
2019-07-07 09:32:50 [19] |    /etc/letsencrypt/live/frdr.ch/fullchain.pem
2019-07-07 09:32:50 [19] |    Your key file has been saved at:
2019-07-07 09:32:50 [19] |    /etc/letsencrypt/live/frdr.ch/privkey.pem
2019-07-07 09:32:50 [19] |    Your cert will expire on 2019-10-05. To obtain a new or tweaked
2019-07-07 09:32:50 [19] |    version of this certificate in the future, simply run certbot
2019-07-07 09:32:50 [19] |    again. To non-interactively renew *all* of your certificates, run
2019-07-07 09:32:50 [19] |    "certbot renew"
2019-07-07 09:32:50 [19] |  - If you like Certbot, please consider supporting our work by:
2019-07-07 09:32:50 [19] |    Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
2019-07-07 09:32:50 [19] |    Donating to EFF:                    https://eff.org/donate-le
2019-07-07 09:32:50 [19] | ### Revoke and delete certificates if needed ####
2019-07-07 09:32:50 [19] | ### Reloading circusd configuration ###
2019-07-07 09:32:51 [19] | ok
```

If you leave the container running, it will check twice a month with _Let's Encrypt_ and renew the certificate if required. Should you need a PFX export of your certificate, set the `PFX_EXPORT=true` and the `PFX_EXPORT_PASSPHRASE="super secret password"` environment variables.

## Certificate Renewal

Since the certificate changes every 3 months with each renewal, I need to find a way to distribute that new certificate to all the places that need it. I am not 100% how to deal with that yet, but I already have some ideas.

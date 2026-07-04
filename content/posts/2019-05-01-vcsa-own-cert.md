---
title: Use your own certificates with vCenter 6.7
date: 2019-05-01T02:24:15
feature_image: https://images.unsplash.com/photo-1553991562-9f24b119ff51?ixlib=rb-1.2.1&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=1080&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ
tags:
  - vmware
  - virtualization
  - tech
languages:
  - en
---

Managing your own CA is usually very tedious and time consuming, so I decided to partly automate it using a Docker container and easy-rsa. The container generates my CA and server/client certificates for all my devices in my home office and the test lab.

VMware vSphere always offered the possibility to use your own certificates, but importing and managing them for all your devices in vCenter was a bit of a horrible experience in version 5.x. With version 6, 6.5 and 6.7 ,the process saw major improvements and the new certificate manager looks very promising. I decided to give it another try!

## Enabling SCP on the vCSA

I always disliked the Windows vCenter, so I was very happy when VMware decided to sunset the Windows version and declared the fairly new vCenter Server Appliance (based on Photon OS) the new standard!

The long-term goal of the appliance is to strip down dead weight, hide everything that is not required for managing your vSphere environment and to use the vCenter API whenever possible. But this also means we need to temporarily enable `scp` and the `bash` shell first:

```shell
» ssh root@vcsa.frdr.ch

VMware vCenter Server Appliance 6.7.0.30000

Type: vCenter Server with an embedded Platform Services Controller

root@vcsa.frdr.ch's password:
Last login: Wed May  1 02:21:46 2019 from 192.168.10.5
Connected to service

    * List APIs: "help api list"
    * List Plugins: "help pi list"
    * Launch BASH: "shell"

Command> shell.set --enable True
Command> shell
Shell access is granted to root
root@vcsa [ ~ ]# chsh -s "/bin/bash" root
root@vcsa [ ~ ]# exit
logout
Command> exit
Connection to vcsa.frdr.ch closed.
```

## Transfer the certificate

Now with `bash` and `scp` available, we can transfer the CA certificate and key to the appliance:

```shell
» scp ca.* root@vcsa.frdr.ch:                                             

VMware vCenter Server Appliance 6.7.0.30000

Type: vCenter Server with an embedded Platform Services Controller

root@vcsa.frdr.ch's password:
ca.crt                                                                         100% 1789   335.6KB/s   00:00
ca.key                                                                         100% 1704   451.3KB/s   00:00
```

Now with all required files in place, we can re-enable `appliancesh`:

```shell
» ssh root@vcsa.frdr.ch

VMware vCenter Server Appliance 6.7.0.30000

Type: vCenter Server with an embedded Platform Services Controller

root@vcsa.frdr.ch's password:
Last login: Wed May  1 02:51:01 2019 from 192.168.10.5
root@vcsa [ ~ ]# chsh -s /bin/appliancesh root
root@vcsa [ ~ ]# exit
logout
Connection to vcsa.frdr.ch closed.
```

## Built-in Certificate Manager

After cert and key are copied, and the appliance shell is reactivated, we can start the certificate-manager and replace the machine certificate:

```shell
root@vcsa [ ~ ]# /usr/lib/vmware-vmca/bin/certificate-manager
 _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _
|                                                                     |
|      *** Welcome to the vSphere 6.7 Certificate Manager  ***        |
|                                                                     |
|                   -- Select Operation --                            |
|                                                                     |
|      1. Replace Machine SSL certificate with Custom Certificate     |
|                                                                     |
|      2. Replace VMCA Root certificate with Custom Signing           |
|         Certificate and replace all Certificates                    |
|                                                                     |
|      3. Replace Machine SSL certificate with VMCA Certificate       |
|                                                                     |
|      4. Regenerate a new VMCA Root Certificate and                  |
|         replace all certificates                                    |
|                                                                     |
|      5. Replace Solution user certificates with                     |
|         Custom Certificate                                          |
|                                                                     |
|      6. Replace Solution user certificates with VMCA certificates   |
|                                                                     |
|      7. Revert last performed operation by re-publishing old        |
|         certificates                                                |
|                                                                     |
|      8. Reset all Certificates                                      |
|_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _|
Note : Use Ctrl-D to exit.
Option[1 to 8]: 2
Do you wish to generate all certificates using configuration file : Option[Y/N] ? : y

Please provide valid SSO and VC privileged user credential to perform certificate operations.
Enter username [Administrator@vsphere.local]:administrator@frdr.ch
Enter password:

Please configure certool.cfg with proper values before proceeding to next step.

Press Enter key to skip optional parameters or use Default value.

Enter proper value for 'Country' [Default value : US] : DE

Enter proper value for 'Name' [Default value : CA] :

Enter proper value for 'Organization' [Default value : VMware] : progressivebytes

Enter proper value for 'OrgUnit' [Default value : VMware Engineering] : Virtualization

Enter proper value for 'State' [Default value : California] : NRW

Enter proper value for 'Locality' [Default value : Palo Alto] : Bochum

Enter proper value for 'IPAddress' (Provide comma separated values for multiple IP addresses) [optional] :

Enter proper value for 'Email' [Default value : email@acme.com] : ops@probytes.eu

Enter proper value for 'Hostname' (Provide comma separated values for multiple Hostname entries) [Enter valid Fully Qualified Domain Name(FQDN), For Example : example.domain.com] : vcsa.frdr.ch

Enter proper value for VMCA 'Name' :progressivebytes
    1. Generate Certificate Signing Request(s) and Key(s) for VMCA Root Signing certificate

    2. Import custom certificate(s) and key(s) to replace existing VMCA Root Signing certificate

Option [1 or 2]: 2

Please provide valid custom certificate for Root.
File : /tmp/ca.crt

Please provide valid custom key for Root.
File : /tmp/ca.key

You are going to replace Root Certificate with custom certificate and regenerate all other certificates
Continue operation : Option[Y/N] ? : y
Get site nameCompleted [Replacing Machine SSL Cert...]
default-site
Lookup all services
Get service default-site:5c26147f-7002-4642-8134-3d6825999e56
Update service default-site:5c26147f-7002-4642-8134-3d6825999e56; spec: /tmp/svcspec_eop8ea2e
Get service default-site:5a5b8cb1-be67-4edf-b72a-4e9fb315c735
Update service default-site:5a5b8cb1-be67-4edf-b72a-4e9fb315c735; spec: /tmp/svcspec_a5c2u08b
Get service default-site:d78475ed-fc75-47b3-8305-7f323e75c272
Update service default-site:d78475ed-fc75-47b3-8305-7f323e75c272; spec: /tmp/svcspec_m8dxysp7
Get service 1c8ca010-aaf6-447a-baa6-3449af72ce0b
Update service 1c8ca010-aaf6-447a-baa6-3449af72ce0b; spec: /tmp/svcspec_j5e_dq95
Get service 49434758-4e1a-4447-8d2f-15e9eae04ce7
Update service 49434758-4e1a-4447-8d2f-15e9eae04ce7; spec: /tmp/svcspec_so1udn8b
Get service 5d00bf72-046e-490a-b4a1-d6f764323377
Update service 5d00bf72-046e-490a-b4a1-d6f764323377; spec: /tmp/svcspec_wvv1ntx8
Get service 76b210b0-b510-4749-80cb-a5fb84443531
Update service 76b210b0-b510-4749-80cb-a5fb84443531; spec: /tmp/svcspec_u019i7l6
Get service dddd359a-2d30-48fd-97e7-f718920f7898
Update service dddd359a-2d30-48fd-97e7-f718920f7898; spec: /tmp/svcspec_pfidwrk4
Get service 9f94d5ad-15e3-41d9-a907-cc23a5b741b4_kv
Update service 9f94d5ad-15e3-41d9-a907-cc23a5b741b4_kv; spec: /tmp/svcspec_82z_zt_1
Get service c3e85b3d-22e6-4519-b528-b3f0245647f7
Update service c3e85b3d-22e6-4519-b528-b3f0245647f7; spec: /tmp/svcspec_iqs4hm3h
Get service 4a68b54d-b633-4d2d-bca0-da6b58f54d33
Update service 4a68b54d-b633-4d2d-bca0-da6b58f54d33; spec: /tmp/svcspec_93p5n2at
Get service 4380648a-70ec-4107-a284-e0e5add24c9f
Update service 4380648a-70ec-4107-a284-e0e5add24c9f; spec: /tmp/svcspec_4ndtjed4
Get service ac3a2e20-5392-457b-86e4-2a8f87d825ea
Update service ac3a2e20-5392-457b-86e4-2a8f87d825ea; spec: /tmp/svcspec_hz707vin
Get service d430e17b-0e9b-498d-bbcb-aaf8bdbed1cf
Update service d430e17b-0e9b-498d-bbcb-aaf8bdbed1cf; spec: /tmp/svcspec_b8cnktdl
Get service 0c16a8c7-205e-40b6-a13b-780d9a072104
Update service 0c16a8c7-205e-40b6-a13b-780d9a072104; spec: /tmp/svcspec_dpwtqfxn
Get service 913c51c1-d64b-4983-861d-c2a97ce3cbbe
Update service 913c51c1-d64b-4983-861d-c2a97ce3cbbe; spec: /tmp/svcspec_bqjcxi57
Get service ecef223a-75e7-4042-8c3c-677b146fb3a7
Update service ecef223a-75e7-4042-8c3c-677b146fb3a7; spec: /tmp/svcspec_imoipa2o
Get service 105fcf9b-cffb-49b0-b524-38acb6d63130
Update service 105fcf9b-cffb-49b0-b524-38acb6d63130; spec: /tmp/svcspec_v8on3sof
Get service bb8aa49b-24f8-4fcc-aba2-fa69662ede09
Update service bb8aa49b-24f8-4fcc-aba2-fa69662ede09; spec: /tmp/svcspec_ojuitv8o
Get service e0c9c6b1-a7a5-4299-982e-50e88d09c95c
Update service e0c9c6b1-a7a5-4299-982e-50e88d09c95c; spec: /tmp/svcspec_3had9glk
Get service ab305cbc-fb09-4f97-94c1-f013518b1674
Update service ab305cbc-fb09-4f97-94c1-f013518b1674; spec: /tmp/svcspec_aua6gm12
Get service b0beee9e-d054-48b6-806c-67026ab90bb2
Update service b0beee9e-d054-48b6-806c-67026ab90bb2; spec: /tmp/svcspec_4buau7vx
Get service f4ffb6b7-3c40-4358-a346-92d4646e8515
Update service f4ffb6b7-3c40-4358-a346-92d4646e8515; spec: /tmp/svcspec_y9lkhuy4
Get service 9f94d5ad-15e3-41d9-a907-cc23a5b741b4
Update service 9f94d5ad-15e3-41d9-a907-cc23a5b741b4; spec: /tmp/svcspec_8_nae0o2
Get service 31a14941-88c4-4689-a548-9f1207525dfb
Update service 31a14941-88c4-4689-a548-9f1207525dfb; spec: /tmp/svcspec_jnb6p56i
Get service 9f94d5ad-15e3-41d9-a907-cc23a5b741b4_authz
Update service 9f94d5ad-15e3-41d9-a907-cc23a5b741b4_authz; spec: /tmp/svcspec_26gbsgtj
Get service 5adc05e4-5fd5-4a36-a54f-47b18ed5c926
Update service 5adc05e4-5fd5-4a36-a54f-47b18ed5c926; spec: /tmp/svcspec_vn8klv1a
Get service 1c8ca010-aaf6-447a-baa6-3449af72ce0b_com.vmware.vsphere.client
Don't update service 1c8ca010-aaf6-447a-baa6-3449af72ce0b_com.vmware.vsphere.client
Get service 9ee61c61-6bd9-4ef4-8a29-1e398087d322
Update service 9ee61c61-6bd9-4ef4-8a29-1e398087d322; spec: /tmp/svcspec_70wdglkf
Get service d860ea63-e0d5-4c8e-a7c9-0cbba669ac20
Update service d860ea63-e0d5-4c8e-a7c9-0cbba669ac20; spec: /tmp/svcspec_hkcdqxwe
Get service 1c8ca010-aaf6-447a-baa6-3449af72ce0b_com.vmware.vsan.dp
Don't update service 1c8ca010-aaf6-447a-baa6-3449af72ce0b_com.vmware.vsan.dp
Get service 24c43f07-476a-43f2-a101-6c295f5ba180
Update service 24c43f07-476a-43f2-a101-6c295f5ba180; spec: /tmp/svcspec__q6y4qg1
Get service 518833e3-bd4f-49c5-895a-84927e6f7e11
Update service 518833e3-bd4f-49c5-895a-84927e6f7e11; spec: /tmp/svcspec_pksb4lms
Get service d5c66c79-9ef1-4bdc-ac9e-2b363b79c0c1
Update service d5c66c79-9ef1-4bdc-ac9e-2b363b79c0c1; spec: /tmp/svcspec_3hz1x1nz
Get service c7dd3576-dd91-4a82-8337-884ca8a53338
Update service c7dd3576-dd91-4a82-8337-884ca8a53338; spec: /tmp/svcspec_b3f34v4c
Updated 33 service(s)
Status : 60% Completed [Replace vpxd-extension Cert...]
2019-05-01T04:49:20  Updating certificate for "com.vmware.imagebuilder" extension

Status : 100% Completed [All tasks completed successfully]
```

After all certificates are re-generated and all services are restarted, the vCenter and all added ESXi hosts will have certificates signed by the CA you uploaded to the vCSA.

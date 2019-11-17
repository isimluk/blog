+++
title = "Steps to set-up Yubikey authentication to FreeIPA"
description = ""
type = ["posts","post"]
tags = [
    "OpenSSL",
    "FreeIPA",
    "YubiKey"
]
date = "2019-11-25"
categories = [
    "OpenSSL",
    "FreeIPA",
    "YubiKey"
]
series = ["Smart Cards"]
[ author ]
  name = "Šimon Lukašík"
+++

This post presents happy path (or reference architecture). The aim is to provide
reproducible steps to configure functioning smart card environment.

There are multiple ways to set-up smart card authentication. Configuration varies based on factors like

 - the CA that signs keys on smart cards
 - properties of CN (Common Name) that are required
 - identity mapping rules (how to translate CN from smart card to identity)
 - versions of client & server stack
 
This blog post gives concrete steps on how to set-up FreeIPA 4.6.6 (Red Hat Identity Management) server for authentication with yubikey smart card.


## Generate private key and CSR inside yubikey

First thing we need is to generate public/private key pair on the smart card. The public
part will then be signed by FreeIPA CA and hence we need to generate CSR (Certificate
Signing Request) along the way.

The easiest and the most approachable way to generate CSR using GUI manager
[yubikey-manager-qt](https://developers.yubico.com/yubikey-manager-qt/). Although there
are other ways to achieve this using either [yubico-piv-tool](https://developers.yubico.com/yubico-piv-tool/), 
[pkcs11-tool](https://github.com/OpenSC/OpenSC/blob/master/src/tools/pkcs11-tool.c), or
[p11tool](https://gnutls.org/manual/html_node/p11tool-Invocation.html)

### Installing YubiKey Manager
Installation of the Yubikey manager is very easy and straight forward. Since we need this program
only for initial settings. We can use latest AppImage from their
[release page](https://developers.yubico.com/yubikey-manager-qt/Releases/).

```
wget https://developers.yubico.com/yubikey-manager-qt/Releases/yubikey-manager-qt-1.1.3-linux.AppImage
chmod u+x yubikey-manager-qt-1.1.3-linux.AppImage
./yubikey-manager-qt-1.1.3-linux.AppImage
```

### Generate CSR

Insert Yubikey and navigate to PIV application in the YubiKey Manager.

![manager](/blog-pics/2019-yubikey_freeipa/01-manager.jpg)

![manager](/blog-pics/2019-yubikey_freeipa/02-piv.jpg)

Go to Certificates and hit **Generate** button.

![manager](/blog-pics/2019-yubikey_freeipa/03-certificates.jpg)

Select *Certificate Signing Request (CSR)* option. This will generate private key on the card
(that never leaves the card) and CSR that you can store in the computer.

![manager](/blog-pics/2019-yubikey_freeipa/04-generate_csr.jpg)

Pick Subject Name for your CSR. It may turn out preferential to choose the same name as your login to the FreeIPA.

![manager](/blog-pics/2019-yubikey_freeipa/05-subject_name.jpg)

Don't forgot to store CSR on the computer. You will later need this CSR to be signed by FreeIPA server CA.

![manager](/blog-pics/2019-yubikey_freeipa/06-generate_csr.jpg)

# Set-up FreeIPA server

Setting is very easy and straight forward. You can either consult 
[Red Hat Indentity Management Documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/linux_domain_identity_authentication_and_policy_guide/sc-web-ui-auth)
for detailed description. Nevertheless, following commands run on freeipa server
will set you up.

```
kinit admin
ipa-advise config-server-for-smart-card-auth > config-server-for-smart-card-auth.sh
chmod u+x config-server-for-smart-card-auth.sh
./config-server-for-smart-card-auth.sh ca.crt.pem
```

## Sign Your CSR by FreeIPA

Log in to the webui as admin user.

![manager](/blog-pics/2019-yubikey_freeipa/07-login.jpg)

Create user used for the authentication (unless already exists)

![manager](/blog-pics/2019-yubikey_freeipa/08-add_user.jpg)

Add new certificate for that user.

![manager](/blog-pics/2019-yubikey_freeipa/09-new_certificate.jpg)

Pass in content of the CSR file you created in previous step.

![manager](/blog-pics/2019-yubikey_freeipa/10-csr.jpg)

Server will sign the CSR and asign resulting certificate to the user.

![manager](/blog-pics/2019-yubikey_freeipa/11-signed.jpg)

Download the user certificate. This is the certificate for the private key you generated on the smart card in previous steps.

![manager](/blog-pics/2019-yubikey_freeipa/12-download.jpg)

## Import Signed Certificate back to the smart card

Choose the same authentication slot that contains the private key.

![manager](/blog-pics/2019-yubikey_freeipa/13-import.jpg)

Import the certificate that was signed by identity server previously.

![manager](/blog-pics/2019-yubikey_freeipa/14-imported.jpg)

## Try logging in

Instead of typing up the login and password, click the blue button *Login Using Certificate*.

![manager](/blog-pics/2019-yubikey_freeipa/15-login_using_cert.jpg)

Pin entry dialog should show up asking you for the smart card pin. In case you don't see pin entry dialog [learn how to diagnose your client system](/posts/2019/11/how-to-debug-smart-card-authentication-client/).

![manager](/blog-pics/2019-yubikey_freeipa/16-pinentry.jpg)

Next dialog that shows up lets you choose certificate to be used for authentication. Since we have only one certificate on the Yubikey only one shows up here.

![manager](/blog-pics/2019-yubikey_freeipa/17-choose.jpg)

Voilà!

![manager](/blog-pics/2019-yubikey_freeipa/18-failed.jpg)

For some reason the authentication failed for me on first try. This may happen to You as well as there is about dozen of components that needs to play in orchestra. Let's see if I can debug my failure. 

## Debug and fix the failure

If you get stuck, I suggest to visit [comprehensive collection of debugging hints](https://floblanc.wordpress.com/2017/06/02/freeipa-troubleshooting-smartcard-authentication/) written by Florence Blanc-Renaud. Although, in my particular case, I have got different type of problem.

I have started debugging with apache error logs. I tailed the logs and re-authenticate, to see the failure in the real-time.

```
tail -f /var/log/httpd/*
==> /var/log/httpd/error_log <==
[Wed Nov 20 14:44:18.041621 2019] [lookup_identity:error] [pid 19766] [client 10.40.204.110:58298] lookup_user_by_certificate failed [dbus_connection_send_with_reply_and_block(org.freedesktop.sssd.infopipe.Users.FindByNameAndCertificate)]: [Permission denied], referer: https://dell-r330-11.testrelm.test/ipa/ui/
[Wed Nov 20 14:44:18.041637 2019] [lookup_identity:error] [pid 19766] [client 10.40.204.110:58298] lookup_user_by_certificate cleared r->user, referer: https://dell-r330-11.testrelm.test/ipa/ui/
[Wed Nov 20 14:44:18.041660 2019] [core:error] [pid 19766] [client 10.40.204.110:58298] AH00027: No authentication done but request not allowed without authentication for /ipa/session/login_x509. Authentication not configured?, referer: https://dell-r330-11.testrelm.test/ipa/ui/
```

It seems that for some reason `mod_lookup_identity` is getting permission denied when accessing D-Bus call. D-Bus calls can be authenticated, so my first idea was to investigate whether `mod_lookup_identity` (apache process, apache user) can access the `org.freedesktop.sssd.infopipe.Users.FindByNameAndCertificate`.

Looking into the `/etc/sssd/sssd.conf`, I can see the following lines:

```
[ifp]
allowed_uids = ipaapi, root
```

It seems that only ipaapi and users are allowed to access the api. Let's add apache user to the list.

```
[ifp]
allowed_uids = ipaapi, root, apache
```

restart sssd

    systemctl restart sssd.service

and return back to web authentication:

![manager](/blog-pics/2019-yubikey_freeipa/15-login_using_cert.jpg)

![manager](/blog-pics/2019-yubikey_freeipa/16-pinentry.jpg)

![manager](/blog-pics/2019-yubikey_freeipa/17-choose.jpg)

Voilà!

![manager](/blog-pics/2019-yubikey_freeipa/19-success.jpg)

*Kudos go to [Jan Pazdziora](https://adelton.com) who discussed the set-up with me several times. Thank You Adelton!*

## Further Learning materials
 - [Debugging Series by Florence Blanc-Renaud](https://floblanc.wordpress.com/2017/06/02/freeipa-troubleshooting-smartcard-authentication/)
 - [mod_lookup_identity by Jan Pazdziora](https://www.adelton.com/apache/mod_lookup_identity/)
 - [FreeIPA: Certificate Identity Mapping](https://www.freeipa.org/page/V4/Certificate_Identity_Mapping)
 - [Red Hat IDM Documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/linux_domain_identity_authentication_and_policy_guide/sc-web-ui-auth)
 - [FreeIPA: Smartcard authentication ipa-advise recipes](https://www.freeipa.org/page/V4/Smartcard_authentication_ipa-advise_recipes)
 - [FreeIPA: User Certificates](https://www.freeipa.org/page/V4/User_Certificates)


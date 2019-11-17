+++
title = "How to distribute SCAP content"
type = ["posts","post"]
tags = [
    "OpenSCAP",
]
date = "2012-05-17"
categories = [
    "OpenSCAP",
]
series = ["OpenSCAP 101"]
[ author ]
  name = "Šimon Lukašík"
+++

  ----

We know where to get an SCAP content and how to scan a machine with Spacewalk. But what is the best way to distribute the content to machines?

The answer is the RPM. (Assuming infrastucture managed by Spacewalk).

If you don't believe me, here is what I think. First of all, we have traditional distribution methods: CD, USB stick, scp, ftp, nfs, but each of them is dependent on something, either on physical acces to machine (CD, USB) or a particular software installed & running (scp, ftp, nfs, ...). Either requirement may not be met, consider a cloud or somewhat secured environment. Another option to use Spacewalk and create a script which downloads the content, and then schedule the script through Spacewalk. Well, the first thing the security department turns off is Spacewalk's remote scripting. Note that neither of these methods handles (by default) versioning, integrity, nor confidentiality.

Further, Spacewalk offers the concept of [configuration files](https://fedorahosted.org/spacewalk/wiki/ConfigurationManagement). It allows you to manage & deploy files on target machine through Spacewalk web interface. It can handle versioning and confidentiality of the content. And the process can be even automated with Spacewalk's API. However, the clients must have rhncfg* packages installed. And it might be again security requirement to not allow remote configuration management.

The RPM, on the other hand, is always present on Fedora-based systems. RPM package can be signed. RPM brings file permission (even 000 if needed). Installed RPM can be verified (rpm -V), which checks the integrity of files using SHA256 and file permissions. Moreover, with RPM you can can bring in (even custom) SELinux context. And the best is the integration of RPM in Spacewalk. For example, with Spacewalk you can schedule following actions: Install my-scap-content package. Execute scan. Verify the package. Remove it again.

I am attaching a template spec file, which can be used for an SCAP content distibution. As an example it builds from USGCB RHEL5 Desktop Baseline: [scap-content-USGCB-rhel5desktop.spec](http://isimluk.fedorapeople.org/sw_openscap/blog/scap-content-USGCB-rhel5desktop.spec).

+++
title = "Public SCAP Content"
type = ["posts","post"]
tags = [
    "OpenSCAP",
]
date = "2012-05-14"
categories = [
    "OpenSCAP",
]
series = ["OpenSCAP 101"]
[ author ]
  name = "Šimon Lukašík"
+++

  ----

As stated [earlier](/posts/2012/05/spacewalk-openscap-compliance-audit-of-enterprise-environments/), writing own SCAP content from scratch is rather hard. One needs to understand at least XCCDF, and often also OVAL layer. On the other hand, customization (tailoring) of existing content is easy. One needs only to review the existing and decide which checks he (or she) wants to include in his (her) security profile, and customize a few variables. There is even opensource graphical tool for tailoring, it is called [scap-workbench](https://github.com/ComplianceAsCode/content).

There are groups publishing their XCCDF content under opensource licences. So anyone may start hacking on top of them.

 - [USGCB for RHEL5 Desktop](http://usgcb.nist.gov/usgcb/rhel_content.html): The United States Government Configuration Baseline, It's the official SCAP content for desktops within federal agencies. It has been developed at NIST which collaborated with DoD and Red Hat, Inc. Unfortunatelly, it turned out, that the content tends to become out of date and some parts of OVAL may not fit nicely to the latest Fedoras.
 - [SCAP Security Guide for RHEL6](https://github.com/ComplianceAsCode/content): It reassumes and extends USGCB, containing profiles for desktop, server, and ftp server. This content is under active community development.
 -  [SCE Community Content](https://fedorahosted.org/sce-community-content/): This content does not use OVAL, instead it contains arbitrary scripts as checking engine. Arbitrary scripts might be no-go for some deployments, on the other hand they allow rapid development which might came handy elsewhere. The content combines rules from STIG, Aqueduct, and Sectool.
 - [Content Based on Sectool](http://git.fedorahosted.org/git/?p=openscap.git;a=tree;f=dist/fedora/sectool-xccdf): and obsoleting it. It's also included in the previous.
 - [OpenSCAP Content for Fedora 14](http://git.fedorahosted.org/git/?p=openscap.git;a=tree;f=dist/fedora):" Exemplary SCAP content created for Fedora 14 and maintained by OpenSCAP community.
 - [OpenSCAP Content for RHEL6](http://git.fedorahosted.org/git/?p=openscap.git;a=tree;f=dist/rhel6)

Please let me know, in case you are aware about another group publishing their XCCDF content. I would like to have them added.


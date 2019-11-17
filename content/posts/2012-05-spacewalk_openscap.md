+++
title = "Spacewalk & OpenSCAP :: Compliance Audit of Enterprise Environments"
type = ["posts","post"]
tags = [
    "OpenSCAP",
    "Satellite",
]
date = "2012-05-03"
categories = [
    "OpenSCAP",
    "Satellite",
]
series = ["OpenSCAP 101"]
[ author ]
  name = "Šimon Lukašík"
+++

  ----

In this blog, I will introduce a new open source solution for compliance checking of big GNU/Linux infrastructures. Wondering what the compliance checking is?

If you run a couple of computer systems to do some job for you, you probably don't want them hacked. Either they store a sensitive data or you simply like their job. You may read a few specially large books about system security. However, even after you hardened your systems, you are not done. You'll need to inspect them regularly. You'll maybe setup agents to guard the setting safety, to detect intrusions, to monitor processes, and to forbid a few bytes here and there. For each category open source has something to offer. But wait! You've just ended-up with another application stack to care of! Good luck with switching vendors. :]

That's why there is open standard at NIST (National Institute of Standards and Technology) called SCAP (Security Content Automation Protocol), which attempts to standardize descriptions of software flaws, vulnerabilities, security configurations, and all other things -- to make compliance checking a bit easier. To use SCAP one needs a two things: a tool which implements the standard and a content which feeds the tool.

A good news. On Fedora hosted there is open source implementation of SCAP, project called [OpenSCAP](http://www.open-scap.org/). It contains a tool which can scan a machine, a workbench which helps with a content tailoring and a sample content. With OpenSCAP one can define his desired security profile and scan his machine. At this point, it's necesarry to admit that there are easier tasks in the world than creating the SCAP content.

So, we have a content (of standardized format) which defines the security configuration and a tool which checks for it. It will work for a while, but how do we ensure inter-machine consistency? Moreover, once something terrible happens, we will need a lot of informations quickly. We will need to compare older scans of the suspect to determine when some change have happened. We will need to compare various scans of different machines to get a clue of attack vector. Then, searching for machine which reports certain configuration may also come in handy.

And that's where the effort to integrate OpenSCAP into Spacewalk emerged. [Spacewalk](http://spacewalkproject.org/) is a complex, web based, systems managing system, designed for GNU/Linux clients with the RPM package manager. Besides the systems, Spacewalk is able to manage packages, errata, package channels, and custom non-rpm content. Spacewalk can administer client system from a source provisioning through a whole life cycle. Spacewalk is able to establish remote commands, remote package management, monitoring, and many other.

With OpenSCAP itegrated to Spacewalk, the compliance checking of whole infrastructure would be much easier. No need for manual intervation, everything could be scheduled & executed automatically. Additionally, if the results are stored in the Spacewalk's database, querying and datamining over archieved results is fun! And since some of the integration work is already done, in the next post, I will show how to scan your systems with OpenSCAP enabled Spacewalk.

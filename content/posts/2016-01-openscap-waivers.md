+++
title = "Four ways to handle waivers with OpenSCAP"
description = " This tutorial describes four approaches how to handle a system that for whatever reason needs to be kept not fully compliant."
type = ["posts","post"]
tags = [
    "OpenSCAP",
    "remediation",
]
date = "2016-01-28"
categories = [
    "OpenSCAP",
    "remediation",
]
series = ["OpenSCAP 101"]
[ author ]
  name = "Šimon Lukašík"
+++

This tutorial offers three approaches how to handle a system scan that for whatever reason is not compliant and it cannot be remediated to compliant state.

I have noticed, that when I run the scan on all of my systems, some of them report findings I cannot remediate. I mean the systems are technically incompliant with the profile, but I need to keep the systems in the incompliant state to avoid breakage of the services there. For instance, on one of my clusters, I have couple messaging brokers. On each broker system, I need to keep `qpid` daemon running, although `scap-security-guide` advises me to turn-it off. I can tell the broker system from any other easily. The broker has `python-acme-broker` package installed on it.

So how do I proceed? Basically there are four options.

 1.  Prepare nothing. Keep the reports coming red. Print-them out. Sign-off the failures. There are highly secured deployments that needs to do this. However, this is maybe not for everyone.

 2.  You can advocate for the waiver support in OpenSCAP tooling. There is waiver support in XCCDF standard and we have it preliminary implementation in OpenSCAP ([examplary usage](https://github.com/OpenSCAP/ruby-openscap/blob/c9db65c19dcbdad75212df86163864f5e02b82b5/test/integration/arf_waiver_test.rb#L26)). The problem is that we do not have nice user interface to include waiver yet. The presentation of the waivers in HTML is done.

 3.  Customize policy for each specific system. [Scap-workbench](http://www.open-scap.org/tools/scap-workbench/) is great tool for customization. The `scap-workbench` produces XCCDF Tailoring files. The process is very [quick](http://www.open-scap.org/resources/documentation/customizing-scap-security-guide-for-your-use-case/). You just need to distribute these Tailoring files along the existing DataStream files. You also need to make sure nobody tampers with the tailoring files once they are distributed to the target systems.

 4.  The last option is to include waivers directly in the policy. This allows you to have single policy/profile for whole infrastructure. In my specific example, I could amend existing DataStream file and make the "Disable qpid" rule apply only on the systems that do not have `python-acme-broker` package installed. This approach to things, however, needs to be applied with security engineering rigor. For example, I risk that someone install the python-acme-broker package on other system to allow qpid daemon run unnoticed. In this case, I am fine accepting the risk.

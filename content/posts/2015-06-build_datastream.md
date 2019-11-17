+++
title = "Solution: Build DataStream from Red Hat CVE stream to automate vulnerability scan with Satellite 6.1"
description = "This tutorial describes four approaches how to handle a system that for whatever reason needs to be kept not fully compliant."
type = ["posts","post"]
tags = [
    "OpenSCAP",
    "vulnerability assessment",
]
date = "2015-06-26"
categories = [
    "OpenSCAP",
    "vulnerability assessment",
]
series = ["OpenSCAP 101"]
[ author ]
  name = "Šimon Lukašík"
+++

  ----

Note as of 2019: Red Hat now publishes native DataStream files. So You no longer need to follow steps in this blog. You can download final files files at [redhat.com](https://www.redhat.com/security/data/metrics/ds/v2/).

  ----


This blog describes how to prepare Red Hat OVAL (RHSA/CVE) content for automated vulnerability audit using Foreman or Red Hat Satellite 6.1.

## Using RHSA stream on single machine

Red Hat publishes OVAL content for assessing vulnerabilities at https://www.redhat.com/security/data/metrics/. The easiest way to audit Red Hat Entrprise Linux for unpatched vulnerabilities is to run following commands:

    $ wget https://www.redhat.com/security/data/oval/com.redhat.rhsa-all.xml.bz2
    $ oscap oval eval com.redhat.rhsa-all.xml.bz2

Note: Older oscap may not support .xml.bz2 natively. Please unzip the file first by running `bunzip2 com.redhat.rhsa-all.xml.bz2`.

## Create RHSA DataStream

Download RHSA stream

    wget http://www.redhat.com/security/data/metrics/com.redhat.rhsa-all.xccdf.xml
    wget https://www.redhat.com/security/data/oval/com.redhat.rhsa-all.xml.bz2
    bunzip2 com.redhat.rhsa-all.xml.bz2

Port XCCDF 1.1 to XCCDF 1.2

    xsltproc --stringparam reverse_DNS com.redhat.rhsa \
        /usr/share/openscap/xsl/xccdf_1.1_to_1.2.xsl \
        com.redhat.rhsa-all.xccdf.xml \
        > com.redhat.rhsa-all.xccdf12.xml

Ensure the newly created file is valid

    oscap xccdf validate com.redhat.rhsa-all.xccdf12.xml

Create DataStream from the downloaded OVAL and from transformed XCCDF

    oscap ds sds-compose com.redhat.rhsa-all.xccdf12.xml NEW-RHSA-DS.XML

Now You have DataStream ready for a distribution using Satellite 6.1+. Learn more about upcomming Satellite 6.1 features at https://www.youtube.com/watch?v=XI1QcQL4qQ8.

## Security compliance automation with Red Hat Satellite - 2015 Red Hat Summit

<iframe width="560" height="315" src="https://www.youtube.com/embed/XI1QcQL4qQ8"
frameborder="0"
allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
allowfullscreen></iframe>

+++
title = "A first SCAP content"
description = ""
type = ["posts","post"]
tags = [
    "OpenSCAP",
]
date = "2012-05-07"
categories = [
    "OpenSCAP",
]
series = ["OpenSCAP 101"]
[ author ]
  name = "Šimon Lukašík"
+++

Before we start scanning we need to prepare a SCAP content. Let's start with a very simple one.

Motivation: Imagine that security specialist decided that in the organization there must not be a single machine storing password hashes in /etc/passwd. All the machines must store hashes in /etc/shadow instead. One would say that such configuration is there by default on any resonable system in 2012, but there is a slight problem with wording. Being default is not the same as being setup. For instance, consider the [bug 818130](https://bugzilla.redhat.com/show_bug.cgi?id=818130), which may cause hashes stored in passwd during the kickstart. And for the record, this bug was found by OpenSCAP scan. :]

A content in this example consist of two part, XCCDF (Extensible Configuration Checklist Description Format) and OVAL (Open Vulnerability And Assesment Language). The OVAL represents the system configuration and the XCCDF builds on top of that and makes a creation of structured security checklist possible. XCCDF, is not utterly needed for our motivation example, but it's esential in more sophisticated scenarios. For example, when user want's to create single file with multiple security profiles.

For start, let's have a look at the checklist (the XCCDF document). It is XML file containing metadata for the check and for the document as whole. The top level element is `<Benchmark>`. Some of its items are compulsory, like document's `<status>` and `<version>` while the most of others are optional.

    <Benchmark xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xmlns="http://checklists.nist.gov/xccdf/1.1" id="first"
               xsi:schemaLocation="http://checklists.nist.gov/xccdf/1.1 xccdf-1.1.4.xsd"
               resolved="false" xml:lang="en-US">
        <status date="2012-05-07">draft</status>
        <title>My First Benchmark</title>
        <version>1.0</version>
    </Benchmark>

All the other elements like `<Rule>`, `<Group>` of rules, and security `<Profile>` are enclosed within the top level `<Benchmark>` element. For our purposes we will need only one `<Rule>` element which defines that there must be no password hash in `/etc/passwd`.  For the `<Rule>` we define id, `<title>`, `<description>`, and `<rationale>`. And also references to another documents: the `<reference>` points out to further reading, the `<ident>` assigns the identifier from configuration enumeration system, and the `<check>` references OVAL definition.

    <Rule id="no_hashes_outside_shadow" selected="true">
      <title>Verify All Account Password Hashes are Shadowed</title>
      <description>To ensure that no password hashes are stored in
        <xhtml:code xmlns:xhtml="http://www.w3.org/1999/xhtml">/etc/passwd</xhtml:code>,
        the following command should have no output:
        <xhtml:pre xmlns:xhtml="http://www.w3.org/1999/xhtml">
        # awk -F: '($2 != "x") {print}' /etc/passwd</xhtml:pre>
      </description>  
      <rationale>
        The hashes for all user account passwords should be stored in
        the file <xhtml:code xmlns:xhtml="http://www.w3.org/1999/xhtml">
        /etc/shadow</xhtml:code> and never in <xhtml:code
        xmlns:xhtml="http://www.w3.org/1999/xhtml">/etc/passwd</xhtml:code>,
        which is readable by all users.
      </rationale>
      <reference href="http://csrc.nist.gov/publications/nistpubs/800-53-Rev3/sp800-53-rev3-final.pdf">
        IA-5
      </reference>
      <ident system="http://cce.mitre.org">CCE-14300-8</ident>
      <check xmlns:xhtml="http://www.w3.org/1999/xhtml"
          system="http://oval.mitre.org/XMLSchema/oval-definitions-5">
        <check-content-ref href="first_oval.xml" name="oval:my_first:def:1"/>
      </check>
    </Rule>

Note that in XCCDF, there is no definition of how to make check, it is in OVAL document and referenced by `oval:my_first:def:1`. The OVAL document will not be discussed, firstly it is not that straight forward, and secondly in next examples we will not need to know it. 

Once we have the content we can scan a machine:

    # yum install openscap-utils -y
    # /usr/bin/oscap xccdf eval first_xccdf.xml

Download full: [first_xccdf.xml](/blog-data/2012-05-first_scap/first_xccdf.xml) and [first_oval.xml](/blog-data/2012-05-first_scap/first_oval.xml)

Credits: The content in the example is based on [SCAP Security Guide](https://github.com/openscap/scap-security-guide) project on Github.

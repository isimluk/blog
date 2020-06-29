+++
title = "Vulnerability scanning in disconnected environments"
description = ""
type = ["posts","post"]
tags = [
    "OpenSCAP",
    "vulnerability assessment",
]
date = "2020-06-29"
categories = [
    "OpenSCAP",
    "vulnerability assessment",
]
series = ["OpenSCAP 101"]
[ author ]
  name = "Šimon Lukašík"
+++

In this post we will learn how to instruct OpenSCAP to build us a DataStream that does vulnerability scanning and yet it does not require internet connection to fetch the latest vulnerability feed.

## Introduction to remote references in SCAP

An XCCDF file may contain references to remote content. That is useful in cases when we want scanners to fetch content from the remote location every time the scan is run. This may be beneficial in cases when the referenced context changes often and we always want to use the latest greatest version for scanning. Widely used example is vulnerability scanning. Consider the following snippet (or similar) that is part of any Red Hat Enterprise Linux guidance.

```
<Rule id="security_patches_up_to_date" selected="false" severity="high">
  <title xml:lang="en-US">Ensure Software Patches Installed</title>
  <ident system="https://nvd.nist.gov/cce/index.cfm">CCE-26895-3</ident>
  <check system="http://oval.mitre.org/XMLSchema/oval-definitions-5">
    <check-content-ref href="https://www.redhat.com/security/data/oval/com.redhat.rhsa-RHEL7.xml"/>
  </check>
</Rule>
```

## How are remote resources processed by OpenSCAP?
OpenSCAP has multiple entry points that allow you to process remote resources. During scanning (`oscap xccdf eval`), OpenSCAP attempts to download the resources locally if `--fetch-remote-resources` is provided.


```
oscap xccdf eval --fetch-remote-resources ssg-rhel7-xccdf.xml 
Downloading: https://www.redhat.com/security/data/oval/com.redhat.rhsa-RHEL7.xml ... ok

(...)
```

If this option is omitted by user, OpenSCAP won't be fetching remote resources and thus the above Rule will be skipped from evaluation. Users may be already familiar with the banner OpenSCAP prints out in this case:
```
$ oscap xccdf eval ssg-rhel7-xccdf.xml 
WARNING: This content points out to the remote resources. Use `--fetch-remote-resources' option to download them.
WARNING: Skipping https://www.redhat.com/security/data/oval/com.redhat.rhsa-RHEL7.xml file which is referenced from XCCDF content

(...)
```

Additionally, you can use `oscap info` command to learn about remote resources ex-ante the scanning.

```
$ oscap info ssg-rhel7-xccdf.xml | grep -A100 Referenced.check.files
Referenced check files:
	ssg-rhel7-ocil.xml
		system: http://scap.nist.gov/schema/ocil/2
	ssg-rhel7-oval.xml
		system: http://oval.mitre.org/XMLSchema/oval-definitions-5
	https://www.redhat.com/security/data/oval/com.redhat.rhsa-RHEL7.xml
		system: http://oval.mitre.org/XMLSchema/oval-definitions-5

```

## Is there a trick to circumvent this behavior?
Absolutely! Within realm of opensource you can always find a subtle little quirk to circumvent or even short-circuit anything.

In case You need to run the vulnerability scan in a disconnected environment, you have to create all-in-one DataStream that contains both the compliance guidance and the vulnerability feed.

First of all you need to get your XCCDF content in a form of SCAP DataStream. [ComplianceASCode](https://github.com/ComplianceAsCode/content) provides well formed SCAP content in various forms, in case you have content from elsewhere chances are you will have to [up your craft to get DataStream](/posts/2013/04/how-to-convert-usgcb-to-datastream-with-openscap/). In this example we will be building DataStream ourselves, but we will assume everything goes well (happy path).

```
# Create SCAP datastream
oscap ds sds-compose ssg-rhel7-xccdf-1.2.xml my_datastream.sds.xml

# Ensure remote resource is referenced, but not listed among checks
$ oscap info my_datastream.sds.xml | grep -A100 Referenced.check
		Referenced check files:
			ssg-rhel7-ocil.xml
				system: http://scap.nist.gov/schema/ocil/2
			ssg-rhel7-oval.xml
				system: http://oval.mitre.org/XMLSchema/oval-definitions-5
			https://www.redhat.com/security/data/oval/com.redhat.rhsa-RHEL7.xml
				system: http://oval.mitre.org/XMLSchema/oval-definitions-5
Checks:
	Ref-Id: scap_org.open-scap_cref_ssg-rhel7-ocil.xml
	Ref-Id: scap_org.open-scap_cref_ssg-rhel7-oval.xml

# Download referenced check
wget 'https://www.redhat.com/security/data/oval/com.redhat.rhsa-RHEL7.xml'

# Add downloaded check to the datastream
oscap ds sds-add com.redhat.rhsa-RHEL7.xml my_datastream.sds.xml

# Verify check is present
$ oscap info my_datastream.sds.xml | grep -A100 Checks:
Checks:
	Ref-Id: scap_org.open-scap_cref_ssg-rhel7-ocil.xml
	Ref-Id: scap_org.open-scap_cref_ssg-rhel7-oval.xml
	Ref-Id: scap_org.open-scap_cref_com.redhat.rhsa-RHEL7.xml
```

At this point we have DataStream that contains original compliance guidance as well as current version of the vulnerability feed. The problem is that there is missing cross reference between these two items. The compliance checklist needs to know to not fetch the remote resource, but use the local one included in the datastream. Easiest way to achieve this is to open DataStream in the text editor and add this mapping manually.

```
<ds:checklists>
  <ds:component-ref id="scap_org.open-scap_cref_ssg-rhel7-xccdf-1.2.xml" xlink:href="#scap_org.open-scap_comp_ssg-rhel7-xccdf-1.2.xml">
    <cat:catalog>
      <cat:uri name="ssg-rhel7-ocil.xml" uri="#scap_org.open-scap_cref_ssg-rhel7-ocil.xml"/>
      <cat:uri name="ssg-rhel7-oval.xml" uri="#scap_org.open-scap_cref_ssg-rhel7-oval.xml"/>

      <!-- The following line was added manually. It translates remote resource to component id within the datastream -->
      <cat:uri name="https://www.redhat.com/security/data/oval/com.redhat.rhsa-RHEL7.xml" uri="#scap_org.open-scap_cref_com.redhat.rhsa-RHEL7.xml"/>

    </cat:catalog>
  </ds:component-ref>
</ds:checklists>
```

Now, you can use this DataStream to run vulnerability & compliance scans in your disconnected environment. Audit, Fix and Be Merry!

**It is highly recommended to re-build such DataStream periodically. Remember the original reason these resources are fetch at the scanning time because they tend to change often. Not re-building content periodically may (and will) result in unpatched vulnerability being undetected.**

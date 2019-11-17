+++
title = "OpenSCAP Remediation"
type = ["posts","post"]
tags = [
    "OpenSCAP",
    "remediation",
]
date = "2013-03-22"
categories = [
    "OpenSCAP",
    "remediation",
]
series = ["OpenSCAP 101"]
[ author ]
  name = "Šimon Lukašík"
+++

  ----

The XCCDF standard provides a vendor neutral way to compose a checklist for assessing computer systems. While the XCCDF specification focuses mainly on the scan, there is also a small module which defines how to put the scanned system into the desired (compliant) state. This blogpost introduces a new feature of OpenSCAP, the remediation, which helps you execute remediation command in the automated way.

Your security policy may contain the remediation commands for each Rule within the XCCDF document. Remediation commands are encapsulated within `<xccdf:fix>` elements. Attributes of the fix element define metadata consumed by scanner tool in the time of execution. For more detailed info about XCCDF Remediation please consult [NIST-IR-7275](http://csrc.nist.gov/publications/PubsNISTIRs.html#NIST-IR-7275-r4), and search for fix and remediation keywords.

The following snippet is an example of a XCCDF Rule with both remediation and scanning (OVAL) definitions. The `@system` attribute defines the interpreter of the commands, you can use bash, perl, python, or whatever your scanner supports. Obviously, the fix content tends to be unportable across different operating systems, thus the XCCDF allows you to specify multiple fix elements -- each for different OS. These fix elements can be distinguished by diffrerent CPE identifier within the `@platform` attribute. The scanner select the one most suitable for the target system. Other attributes like, `@disruption` level or flag indicating whether `@reboot` is required, can be used by scanner tool to choose the fix element.

    <Rule id="xccdf_moc.elpmaxe.www_rule_1">
      <title> The telnet-server Package Shall Not Be Installed </title>
      <rationale>
        Removing the telnet-server package decreases the risk
        of the telnet service's accidental (or intentional) activation.
      </rationale>
      <fix platform="cpe:/o:redhat:enterprise_linux:6" reboot="false"
        disruption="low" system="urn:xccdf:fix:script:sh">
        if rpm -q telnet-server; then
            yum -y remove telnet-server
        fi
      </fix>
      <check system="http://oval.mitre.org/XMLSchema/oval-definitions-5">
        <check-content-ref
            href="test_remediation_simple.oval.xml"
            name="oval:moc.elpmaxe.www:def:1"/>
      </check>
    </Rule>

## Remediation Support in Earlier OpenSCAP

Previously, in OpenSCAP we didn't execute fix elements. We have only allowed export of bash fix scripts to a file. In order to bring your machine to the compliant state, You have need to take the following four steps:

 - Execute OpenSCAP scan and get `xccdf:TestResult` stored to some file like:
    ```
    $ oscap xccdf eval \
        --results ~/my-results-xccdf.xml \
        /usr/share/scap/my-policy-xccdf.xml
    ```

 - Generate bash script from the result file:
    ```
    $ oscap xccdf generate fix \
        --output ~/my-bash-remediation.sh \
        ~/my-results-xccdf.xml
    ```

 - Review the script by eye and execute it.
    ```
    $ ~/my-bash-remediation.sh
    ```

 - Execute OpenSCAP scan again and verify applied fixes:
    ```
    $ oscap xccdf eval \
        --results ~/my-results-xccdf2.xml \
        /usr/share/scap/my-policy-xccdf.xml
    ```

### Shortcomings of That Approach
For the second step, OpenSCAP library used XSLT transformation to generate scripts. However, some of the features of XCCDF (Text Substitution, CPE applicability, and correct XCCDF Processing) were missing and these features seem to be bordeline impossible to implement with pure XSLT language. Further, when the administrator executed the remediation script manually in the third step, the output of the script was not saved back to the XCCDF document and the second scan may not be issued. Thus the chain of evidences was discontinued.

The newer version of OpenSCAP library addresses these problems by integrating these 4 steps together. It introduces two new processess which I named Online and Offline remediation.

## oscap's Online Remediation
The online remediation executes `fix` elements in the time of scanning. First of all OpenSCAP evaluates XCCDF as usual, assessing the results by evaluation of OVAL definitions. Next, each Rule which has failed is candidate for remediation. OpenSCAP searches for appropriate fix element, resolves it, prepares the environment and executes the script. Any output of the fix script is captured by OpenSCAP and stored within the result element. The return value of the fix script is stored as well.

Whenever OpenSCAP executes fix script, it immediatelly evaluates the OVAL definition again (to verify that fix script has been applied correctly). In this second run, if the OVAL returns success the result of the Rule is `fixed` otherwise it is `error`. The online remediation is executed by `--remediate` command-line switch.


    $ oscap xccdf eval --remediate \
        --results ~/my-results-xccdf.xml \
        /usr/share/scap/my-policy-xccdf.xml

## oscap's Offline Remediation
The offline remediation on the other hand allows you to postpone fix execution. In first step, you only evaluate system and store the `xccdf:TestResult` on your disk:

    $ oscap xccdf eval \
        --result ~/my-results-xccdf.xml \
        /usr/share/scap/my-policy-xccdf.xml

In the second step, oscap executes the fix scripts and verifies the result. It is safe to store results into the input file, no data will be lost. During the offline remediation OpenSCAP creates new `xccdf:TestResult` element which is based on the previous one and inherits all the data. The newly created TestResult differs only in the rule-result elements which have had failed in the first place. For those the remediation is executed.


    $ oscap xccdf remediate \
        --results ~/my-results-xccdf.xml \
        ~/my-results-xccdf.xml

## Examplary result XCCDF
The following are snippets of the `~/my-results-xccdf.xml`. These demostrate what data are exported by OpenSCAP to the result document. In the first example the rule-result failed to remediate, because I had not enough privileges to modify package set on the system.

```
<rule-result idref="xccdf_moc.elpmaxe.www_rule_1" time="2013-03-22T19:15:11" weight="1.000000">
  <result>error</result>
  <message severity="info">Fix execution comleted and returned: 1</message>
  <message severity="info">Loaded plugins: auto-update-debuginfo, langpacks, presto, refresh-packagekit
You need to be root to perform this command.
  </message>
  <message severity="info">Failed to verify applied fix: Checking engine returns: fail</message>
  <fix xmlns:xhtml="http://www.w3.org/1999/xhtml" platform="cpe:/o:redhat:enterprise_linux:6"
    reboot="false" disruption="low" system="urn:xccdf:fix:script:sh">
    if rpm -q telnet-server; then
        yum remove -y telnet-server
    fi
  </fix>
  <check system="http://oval.mitre.org/XMLSchema/oval-definitions-5">
    <check-content-ref name="oval:moc.elpmaxe.www:def:1" href="test_remediation_simple.oval.xml"/>
  </check>
</rule-result>
```

In the second try, I logged in as root user and the remediation succeeded. You can see that OpenSCAP exported full output of the fix script. The output of yum command within the XML file looks clumsy. On the other hand, I believe that in the critical environments the output of remediation command should be properly archieved, especially if the remediation is fully automated.

```
<rule-result idref="xccdf_moc.elpmaxe.www_rule_1" time="2013-03-22T19:16:03" weight="1.000000">
      <result>fixed</result>
      <message severity="info">Fix execution comleted and returned: 0</message>
      <message severity="info">Loaded plugins: auto-update-debuginfo, langpacks, presto, refresh-packagekit
Resolving Dependencies
--&gt; Running transaction check
---&gt; Package telnet-server.x86_64 1:0.17-51.fc16 will be erased
--&gt; Finished Dependency Resolution

Dependencies Resolved

================================================================================
Package              Arch          Version                Repository      Size
================================================================================
Removing:
telnet-server        x86_64        1:0.17-51.fc16        @fedora        53 k

Transaction Summary
================================================================================
Remove  1 Package

Installed size: 53 k
Downloading Packages:
Running Transaction Check
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Erasing    : 1:telnet-server-0.17-51.fc16.x86_64                          1/1
  Verifying  : 1:telnet-server-0.17-51.fc16.x86_64                          1/1

Removed:
  telnet-server.x86_64 1:0.17-51.fc16

Complete!
</message>
      <fix xmlns:xhtml="http://www.w3.org/1999/xhtml" platform="cpe:/o:redhat:enterprise_linux:6"
        reboot="false" disruption="low" system="urn:xccdf:fix:script:sh">
        if rpm -q telnet-server; then
            yum remove -y telnet-server
        fi
    </fix>
      <check system="http://oval.mitre.org/XMLSchema/oval-definitions-5">
        <check-content-ref name="oval:moc.elpmaxe.www:def:1" href="test_remediation_simple.oval.xml"/>
      </check>
    </rule-result>
```

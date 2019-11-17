+++
title = "Spacewalk & OpenSCAP :: Detailed SCAP Results"
type = ["posts","post"]
tags = [
    "OpenSCAP",
    "Satellite",
]
date = "2013-08-07"
categories = [
    "OpenSCAP",
    "Satellite",
]
series = ["OpenSCAP 101"]
[ author ]
  name = "Šimon Lukašík"
+++

  ----

This blog post describes a new feature of Spacewalk 2.0 related to OpenSCAP integration.

In previous versions, Spacewalk was able to schedule OpenSCAP scan on the managed systems and gather SCAP results to its database. Only a brief sub-set of SCAP results was stored in Spacewalk database. That included the result of evaluation for each `xccdf:Rule`, rule's ID, and CVE or CCE identifiers assigned to it. This kind of information was essential to spot misconfiguration early, however it was often not sufficient to investigate the causes of the failure. The more comprehensive information was available to a user only after manual scan re-run.

Since there were users requesting for Spacewalk to include more SCAP information, I decided to extend amount of aggregated data. Spacewalk 2.0 is now able to grab and store SCAP result files produced by OpenSCAP scanner. The figure below shows how the new data is presented.

![detailed results](/blog-pics/2013-detailed_results/blog-13-detailed-results.jpg)

The scan's details page now includes **Detailed Results** row which enables user to download SCAP Result files. There are three types of SCAP results files:

 - `xccdf-result.xml` - The XCCDF Result File as described by SCAP standard.

 - `*.result.xml` - The OVAL Result Files as described by SCAP standard. There will be one such file for each OVAL Definition file on input.

 - `xccdf-report.html` - The human readable report for the scan. It includes summary information from XCCDF and OVAL files.

The above figure shows details of SSG (SCAP Security Guide) evaluation. There is the XCCDF result file, the HTML report and the ssg-rhel6-oval.xml.result.xml representing OVAL results. All the files can be downloaded using web browser.

## Setting This Up

By default Spacewalk does not aggregate detailed SCAP results. Please follow these two steps to set-up detailed SCAP results add-on.


 - On the client side: Make sure you have spacewalk-oscap package of version greater or equal to 0.0.15.

 - On the server side: Make sure you have enabled SCAP Detailed Results in the organization configuration dialog.

    The organization configuration dialog (figure below) contains two rows related to this feature. The first is **Enable Upload Of Detailed SCAP Files** which turns the feature on. The second, **SCAP File Upload Size Limit**, defines the biggest acceptable size of file in Bytes. The size limit defaults to 2 MiB.

    ![organization settings](/blog-pics/2013-detailed_results/blog-14-org-config.jpg)

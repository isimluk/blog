+++
title = "Spacewalk & OpenSCAP :: Deletion of SCAP Results"
type = ["posts","post"]
tags = [
    "OpenSCAP",
    "Satellite",
]
date = "2013-08-08"
categories = [
    "OpenSCAP",
    "Satellite",
]
series = ["OpenSCAP 101"]
[ author ]
  name = "Šimon Lukašík"
+++

  ----

This blog post describes a new feature of Spacewalk 2.1 (not yet released) which allows for SCAP results at the server to be deleted.

Previously, it was impossible to delete an SCAP scan from Spacewalk, but it turned out that some of the SCAP results loose importance over time and users want to delete them. In general, deletion of auditing results is tricky. The audit may serve as an evidence or may be subject to retention policies. Thus, Spacewalk should not allow SCAP result deletion, unless certain requirements are met.

### Enabling SCAP Result Deletion

By default Spacewalk 2.1 does not allow SCAP result deletion, until the deletion is configured in Organization Configuration dialog.

![organization settings](/blog-pics/2013-detailed_results/blog-14-org-config.jpg)

In the Organization Configuration dialog (shown in figure above), there are two configuration items related to the SCAP Deletion. The first, **Allow Deletion of SCAP results**, enables deletion for the given organization. This option defaults to off, once enabled it allows system administrators to delete any SCAP results they can see and which has already passed its retention period. The second configuration item, **Allow Deletion After**, defines the retention period (in days) for SCAP result. If the retention period is set to zero, SCAP results can be deleted immediately upon finish.

Once configured, SCAP scan results may be deleted on web user interface or through API (using `system.scap.deleteXccdfScan()` function). 

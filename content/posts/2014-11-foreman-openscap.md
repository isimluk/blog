+++
title = "Foreman & OpenSCAP :: First UI Mock-ups"
type = ["posts","post"]
tags = [
    "OpenSCAP",
    "Satellite",
]
date = "2014-11-06"
categories = [
    "OpenSCAP",
    "Satellite",
]
series = ["OpenSCAP 101"]
[ author ]
  name = "Šimon Lukašík"
+++

  ----

In the past few months, I have been busy integrating OpenSCAP to Foreman. Let's see how the user interface of Foreman & OpenSCAP integration may look like.

We will heavily use word "Compliance" within the UI. It seems to more comprehandable than just OpenSCAP. The foreman_openscap plug-in brings in two main concepts:


 - Definition of comliance policy and

 - compliance report of particular asset.


Each of those gets one link from main hosts menu:

![menu](/blog-pics/2014-foreman/blog-01-compliance_menu.jpg)

Similarly, foreman_openscap will introduce two new roles to Foreman's users: can edit compliance policies and can view compliance reports.

![roles](/blog-pics/2014-foreman/blog-06-roles.jpg)

Compliance policies can be listed:

![policies](/blog-pics/2014-foreman/blog-02-list_policies.jpg)

and created or edited by user

![create](/blog-pics/2014-foreman/blog-03-new_policy_1.jpg)

![create](/blog-pics/2014-foreman/blog-03-new_policy_2.jpg)

Once the reports are collected from infrastructure according to the policies definition they can be listed and searched.

![create](/blog-pics/2014-foreman/blog-04-art_reports.jpg)

For each report there are a very verbose detailes available.

![create](/blog-pics/2014-foreman/blog-05-arf_report.jpg)

+++
title = "An XCCDF trap into which authors often fell"
type = ["posts","post"]
tags = [
    "OpenSCAP",
]
date = "2012-12-03"
categories = [
    "OpenSCAP",
]
series = ["OpenSCAP 101"]
[ author ]
  name = "Šimon Lukašík"
+++

  ----

I have seen enough SCAP contents and products (even the official ones) which contain small deviations from the standard. Today I would like to describe one bug which is the most common as well as it is subtle and hard to spot.

In one of my previous [posts](http://localhost:1313/posts/2012/05/a-first-scap-content/), I have shown a very simple XCCDF file, it contained only a single rule. If you extend it by adding similar rules you will end up with a single checklist. What is often needed by organizations are multiple checklist, indeed a different policy you will enforce on personal laptop compared to mission critical server. The XCCDF standard addresses this demand and enables you to carry multiple checklists together in a single file. This is achieved by profiles. Each profile may select a different set of rules in document and thus establish a different policy.

This brings us to the first problem, selecting is complex mechanism and selection of a given rule is determined by multiple factors. These factors needs to be applied in correct order as the standard defines. (By the way that's also the reason why I have recently rewritten selector processing in OpenSCAP). Whether an item becomes selected or not is determined by the following factors:


 - default setting of the item (the @selected attribute)
 - settings in the profile or inherited profiles
 - selection of parent group
 - selection of clusters
 - requires/conflicts dependencies.

To make it even more difficult, the [XCCDF standard](http://csrc.nist.gov/publications/nistir/ir7275-rev4/nistir-7275r4_updated-march-2012_clean.pdf) specifies:

*An unselected group SHALL NOT be processed, and its contents SHALL NOT be processed either (i.e., all descendants of an unselected group are implicitly unselected).*

Which meens that some of your rules might become unselected when the superior group is unselected. This behaviour seems to be inconsistent with what an average content author expects. And even experts might get [confused](https://www.redhat.com/archives/open-scap-list/2012-November/msg00032.html) by this rule. Related problems can be even found in USGCB and STIG guidances.

Hence, my humble advice for content authors is: Please consult your checklist with OpenSCAP scanner and make sure that your final policies really enforce what you expect them to enforce. 

+++
title = "Spacwalk & OpenSCAP :: Search"
type = ["posts","post"]
tags = [
    "OpenSCAP",
    "Satelite",
]
date = "2012-06-18"
categories = [
    "OpenSCAP",
    "Satellite",
]
series = ["OpenSCAP 101"]
[ author ]
  name = "Šimon Lukašík"
+++

  ----

Another SCAP feature has landed in Spacewalk-nightly. It is a web page that allows users to search scan results. User can search xccdf:rules(-s), by their text fields (fulltext), by the results of evaluation, by the time, and/or by the systems.

![search form](/blog-pics/2012-search/blog-06-search_free_form.jpg)

Let's take a look how a complex query could look like. For example, I want to search systems that recently reported that they don't have boot loader locked by password. For this purpose I need the content, which checks for such configuration option and the systems my Spacewalk has scanned.

From the [contents](/posts/2012/05/public-scap-content/) I have learned, that XCCDF rule for boot loader passwords are regarded by **CCE-1818-2** identifier, so I put this down to the text field of the search form. Next, I want to see systems which are not compliant with this rule, so from the drop-down box I choose the **fail** result.

The third selection determines whether to search scans on all the systems, or on a subset only. Concept for managing subset of systems is called [System Set Manager (SSM)](http://docs.redhat.com/docs/en-US/Red_Hat_Network_Satellite/5.4/html/Reference_Guide/sect-Reference_Guide-Systems.html#sect-Reference_Guide-Systems-SSM_mdash_). And since I actually do want to search only a few systems which I've selected afore, I choose to search within the SSM. Bellow, I specify date range: since the beggining of the month up to now. And finally the last choise I leave unchanged.

![search](/blog-pics/2012-search/blog-07-search_ruleresults.jpg)

The search shows that my Spacewalk keeps track of two evaluations (rule-results) of CCE-1818-2 which match given criteria and differs in the idref field (XCCDF Rule Identifier). So far so good, but I get a list of rule-results, while I would much rather get a list of systems. Hence, I change the last search choise to **List of XCCDF Scans**, which will search XCCDF TestResults instead. Only those TestResults are returned which contain rule-results returned by previous query.

![search](/blog-pics/2012-search/blog-08-search_testresults.jpg)

Now, I can see that two different machines fails to have the boot loader locked. I'll fix them right after I'll have coffee.

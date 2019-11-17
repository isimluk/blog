+++
title = "Introducing project SCAPtimony"
type = ["posts","post"]
tags = [
    "OpenSCAP",
    "Satellite",
]
date = "2014-10-31"
categories = [
    "OpenSCAP",
    "Satellite",
]
series = ["OpenSCAP 101"]
[ author ]
  name = "Šimon Lukašík"
+++

  ----

How do I archive all the SCAP result coming from my infrastructure? For how long? What kind of SCAP result post-processing would help me retain control over environment? How do I ensure that all the nodes in given perimeter has been audited by given policy in last week? What are the good practices for operating SCAP audit of multiple nodes? It is a a heck of a lot of XML files and there has to be a better way!

These are common question amongst operational guys and there needs to be piece of software to help answer them. Let me introduce project SCAPtimony, its motives and mission statements.

SCAPtimony project gives full testimony about compliance of your infrastructure. SCAPtimony is open source compliance center build on top of SCAP, the U.S. Government standard. SCAPtimony is a collection (database) of auditable assets, SCAP policies, audit schedules, SCAP results, and waivers. SCAPtimony is modern, RESTful, highly efficient, robust, and cloud-class scalable solution to the common problem of SCAP document storage. Going forward, SCAPtimony pushes the envolope by leveraging OpenSCAP to empower administrators in a sustainable way! ... Bingo!

## Planned Features
+ Define security/compliance policies
    + Archive distinct versions of the policy
    + Upload SCAP content and assign it with the policy
    + Set-up a periodical schedule of audits for the policy
    + Organization defined targeting (Assign a set of nodes with the policy)
    + Define known-issues and waivers (Assign waivers with a set of nodes and the policy)
    + Set-up rules for automated deletion of SCAP results
+ Achieve SCAP audit results from your infrastructure
    + Provide API for tools to upload collected SCAP results
+ Result post-processing
    + Search SCAP results
    + Search for non-compliant systems
    + Search for not audited systems
    + Comparison of audit results
    + Waive known issues

Let me know, if your feature is missing. In the meantime, source codes are brewing at https://github.com/OpenSCAP/scaptimony.

And by the way, project SCAPtimony would never be possible if there was no [oscap_source redesign in OpenSCAP](/posts/2014/10/the-oscap_source-api-redesign/). That redesign significantly improved post-processing capabilities of OpenSCAP needed especialy for SCAPtimony's waivers. 

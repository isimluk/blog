+++
title = "Foreman & OpenSCAP :: Architecture"
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

This blog post is a reminiscence of [my very first post](/posts/2012/05/spacewalk-openscap-compliance-audit-of-enterprise-environments/). Also, this is a follow-up on recent [introduction of project SCAPtimony](/posts/2014/10/introducing-project-scaptimony/) post. First application of SCAPtimony will be integration with Foreman.

[Foreman](http://theforeman.org/) is a complex, web based, systems managing system. It provides life-cycle management as well as configuration management using popular Puppet project.

The integration of Foreman and OpenSCAP will enable administrators to set-up compliance policies for their infrastructure and to audit on continuous basis. The integration effort includes multiple sub-projects. Each of the project has restricted set of responsibilities as follows:


 - [OpenSCAP](http://open-scap.org/page/Main_Page) is open-source implementation of SCAP line of standards. Foreman will leverage broad API of libopenscap.so for manipulation of SCAP files (parsing, modification, creation of HTML reports, etc.).

 - [ruby-openscap](https://github.com/OpenSCAP/ruby-openscap) provides Ruby bindings to OpenSCAP. Or more accurately, ryby-openscap exposes certain OpenSCAP functions to Ruby language using FFI. Since the API of libopenscap.so is quite low-level, ruby-openscap encapsulate data and functions into classes that relate the objects in SCAP standards. Ruby-openscap is still quite light-weight and includes only subset of functionality available in base OpenSCAP package.

 - [puppet-openscap](https://github.com/OpenSCAP/puppet-openscap) exposes SCAP scan operation to Puppet language. With puppet-openscap users can write puppet-modules that will create OpenSCAP scan reports on periodic basis. puppet-openscap also includes a Puppet class that will upload SCAP result immediately to foreman-proxy_openscap.

 - [foreman-proxy_openscap](https://github.com/OpenSCAP/foreman-proxy_openscap) is a tiny plug-in for Foreman's smart-proxy. This plug-in exposes single API function for client system: (PUT /openscap/arf/arf/:policy_name/:date) . This is API is approached by puppet-openscap when uploading SCAP report. Client upload is authenticated using Puppet certificates. Then, the collection of SCAP reports is forwarded to foreman_openscap.

 - [foreman_openscap](https://github.com/OpenSCAP/foreman_openscap) is a Foreman plug-in (rails engine) that binds Foreman and SCAPtimony together. It provides back-end API for foreman-proxy_openscap to upload SCAP reports. Foreman_openscap also provides [user interface](/posts/2014/11/foreman-openscap-first-ui-mock-ups/). Front-end API for user automation is planned as well.

 - [SCAPtimony](https://github.com/OpenSCAP/scaptimony) project has been discussed in [recent blog post](/posts/2014/10/introducing-project-scaptimony/). SCAPtimony is a mountable rails engine. SCAPtimony is independent of Foreman, however it can be mounted to Foreman. SCAPtimony will enable advanced SCAP operation that cannot be implemented in OpenSCAP package, as they need multiple inputs.

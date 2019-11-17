+++
title = "How does OpenSCAP work?"
type = ["posts","post"]
tags = [
    "OpenSCAP",
]
date = "2014-09-11"
categories = [
    "OpenSCAP",
]
series = ["OpenSCAP 101"]
[ author ]
  name = "Šimon Lukašík"
+++

  ----

This is an introductionary into OpenSCAP internals. After read, you will understand the high-level architecture of OpenSCAP.

## Structure of codebase

OpenSCAP project consists of three notable parts: command-line tool, shared library, and the OVAL probes. Let's first take closer look on command-line tool. The oscap is just small binary which makes functionality of the shared available to an user. We are trying to keep the [codebase](https://github.com/OpenSCAP/openscap/tree/master/utils) of oscap command as small as possible.

Everything oscap does is backed by the shared library, `libopenscap.so`. The library is logically divided to [parts](https://github.com/OpenSCAP/openscap/tree/master/src) (XCCDF, OVAL, DataStream, CPE, ...). The parts of the library match to the components of SCAP standard. Each part includes parser and exporter code as well as the implementation of algorithms defined by standard.

The heart of the library is implementation of OVAL language. That is the part that includes a lot of low-level code to query system characteristics. Library uses special executables: OpenSCAP [probes](https://github.com/OpenSCAP/openscap/tree/master/src/OVAL/probes) to examine the system. None of the checks is implemented by library itself.

When a library evaluates OVAL object, it finds an appropriate probe, executes it and sends the query to the probe's stdin. The probe queries system and returns a result on its stdout. The very compact protocol ([SEXP](http://en.wikipedia.org/wiki/S-expression) is used for communication. Each probe holds a cache to minimize system load and avoid duplicate queries.

## Introduction of High-level API

In past, we focused to build a low-level API. That means that users of library `libopenscap.so` are able to build tools on top of the library that may do pretty much everything. For instance an SCAP editor may be build on top of `libopenscap.so`.

On the other hand, when we had this low-level API, it required profound intellectual exercise to write tools on top of it. For example writing simple scanner in C required several hundreds lines of code. That all just to load a datastream, initialize various structures, start evaluation, and export results correctly. There were simply too many options available with the low-level API.

Then, we have introduced [xccdf_session](https://github.com/OpenSCAP/openscap/blob/master/src/XCCDF/public/xccdf_session.h) interface. The xccdf_session interface is recommend starting point for any developer. It allows you to write very flexible XCCDF scanner, despite being abstracted from all low-level decisions.

## Writing simple scanner

The ruby bindings for the library moved this to next level, a scanner tool can be written in 8 easy lines:

    1:     require 'openscap'
    2:     s = OpenSCAP::Xccdf::Session.new("/usr/share/xml/scap/ssg/fedora/ssg-fedora-ds.xml")
    3:     s.load
    4:     s.profile = "xccdf_org.ssgproject.content_profile_common"
    5:     s.evaluate
    6:     s.remediate
    7:     s.export_results(:rds_file => "results.rds.xml")
    8:     s.destroy

All the magic happens on the fifth line of code. The XCCDF module will build a structure called XCCDF_POLICY that holds an evaluation context. That is the object model of XCCDF file, selected profile, resolved values and handles to OVAL files referenced from XCCDF.

The XCCDF Policy then goes through the tree of XCCDF Rules and asks OVAL module to evaluate particular definitions. For each query OVAL module ask particular probe and caches result in form of collected object and module then computes test-result and definition-result. The final value is returned to XCCDF module and forms `xccdf:rule-result`.

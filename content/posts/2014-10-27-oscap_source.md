+++
title = "The oscap_source API redesign"
type = ["posts","post"]
tags = [
    "OpenSCAP",
    "Satellite",
]
date = "2014-10-27"
categories = [
    "OpenSCAP",
    "Satellite",
]
series = ["OpenSCAP 101"]
[ author ]
  name = "Šimon Lukašík"
+++

  ----

This blog is concerning API redesign in libopenscap, its motives, and implications. It is rather significant change coming to OpenSCAP 1.2.0, though it is expected to come unnoticed by OpenSCAP users. Let me just start with historical background.

Prior the introduction of the DataStream file format to SCAP version 1.2, there has been a jungle of distinct file formats in the SCAP world. The OpenSCAP has implemented the very most of them, however common bind between them has been utterly missing.

Hence, we have ended-up with a dozen of independent file parsers. Each parser carried its own structures and its own ways to approach things. Any generic routines that would exist took in the path to file. The processing of SCAP content took us subsequent multiple file openings and multiple initializations of DOM or xmlReader parsers.

When the DataStream file format was introduced to the standard, OpenSCAP followed the easiest path to implement it. That means, we have just created very thin layer that was just decomposing a DataStream file to multiple other SCAP files. We have spin the DataStream support in OpenSCAP quickly and the most easy use-cases were covered. Remember that nobody has been using DataStreams at that time, so we didn't know if DataStreams will be adopted, neither we could anticipate what would be more advanced use-cases. OpenSCAP slitted the DataStream internally to a temporally directory and then we used the old file-openings parser to parse the content of DataStream. Later, we started to be unhappy with this approach, it was hard to add new functionality, the advanced functions for DataStream handling.

This September we have reworked whole file handling. We have introduced [oscap_source](https://github.com/OpenSCAP/openscap/blob/master/src/source/public/oscap_source.h) structure as the abstract handle to any SCAP content. The `oscap_source` abstracts all the parsers from the medium, be it either file, HTTP response or part of another SCAP content (DataStream).

The `oscap_source` also abstracts from common operations one may want to call over the content. The `oscap_source` can tell the document type and schema version, or validate the document. Overall the code is a bit more cleaner, efficient, and flexible.

We have also introduced [source DataStream session](https://github.com/OpenSCAP/openscap/blob/master/src/DS/public/ds_sds_session.h) and [result DataStream session](https://github.com/OpenSCAP/openscap/blob/master/src/DS/public/ds_rds_session.h). These structures hold internal information about the opened DataStream and they can return parts of DataStream in a form of another `oscap_source`.

The change involved more than 300 commits. During the work we have modified each parser to bind well with oscap_source and old the routines were deprecated. While OpenSCAP may carry the deprecated routines working for a while, library users are advised to upgrade to a new API. Each deprecated function points to the new and preferred function.

OpenSCAP is now able to work with DataStreams natively and efficiently without creating temporally files. And as a bonus, we have introduced native support for bzipped files. If the filename is *.xml.bz2, OpenSCAP will recognize the file and process it as a plain XML. That is pretty cool, because it will allow us to build an efficient SCAP results storage, the project [SCAPtimony](https://github.com/OpenSCAP/scaptimony). 

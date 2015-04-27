---
layout: page
title: Software
tags: []
modified: 2014-08-08T20:53:07.573882-04:00
comments: true
share: false
image:
  feature: sample-image-1.jpg
---

### Ocelotl

#### Multidimensional Overviews for Huge Trace Analysis (main developper)

Ocelotl is an innovative visualization tool, which provides overviews for execution trace analysis by using a data aggregation technique. This technique enables to find anomalies in huge traces containing up to several billions of events, while keeping a fast computation time and providing a simple representation that does not overload the user.
Ocelotl is integrated into Framesoc, a generic trace management and analysis infrastructure. You can take advantage of the tool bunch provided by Framesoc, and switch from an Ocelotl's overview to more detailed representations once you know where to focus.

##### Ocelotl's web site: [http://soctrace-inria.github.io/ocelotl/](http://soctrace-inria.github.io/ocelotl/)

### LPAggreg library (main developper)

LPAggreg is a library written in C++ and that enables to aggregate multidimensional and discrete generic systems over one or several dimensions.
It is built upon an aggregation method design by Robin Lamarche-Perrin.

##### LPaggreg's github repository: [https://github.com/dosimont/lpaggreg](https://github.com/dosimont/lpaggreg)

##### JNI wrapper github repository: [https://github.com/dosimont/lpaggregjni](https://github.com/dosimont/lpaggregjni)

### Framesoc (contributor)

Framesoc is the core software infrastructure of the SoC-Trace project. It provides a graphical user environment for execution-trace analysis, featuring interactive analysis views as Gantt charts or statistics views. It provides also a software library to store generic trace data, play with them, and build other analysis tools.

##### Framesoc's web site: [http://soctrace-inria.github.io/framesoc/](http://soctrace-inria.github.io/framesoc/)

---
layout: page
title: Software
tags: []
modified: 2014-08-08T20:53:07.573882-04:00
comments: true
image:
  feature: sample-image-1.jpg
---

## [Ocelotl](http://soctrace-inria.github.io/ocelotl/) 

### Lead Developper

Ocelotl is an innovative visualization tool, which provides overviews for execution trace analysis by using a data aggregation technique. This technique enables to find anomalies in huge traces containing up to several billions of events, while keeping a fast computation time and providing a simple representation that does not overload the user.
Ocelotl is integrated into Framesoc, a generic trace management and analysis infrastructure. This enables to interact with other analysis tools, in particular to get more details on problematic behaviors detected by Ocelotl.

![Ocelotl](/images/ocelotl.png)  

## LPAggreg library

### Lead Developper

LPAggreg is a library written in C++ and that enables to aggregate multidimensional and discrete generic systems over one or several dimensions.
It is built upon an aggregation method designed by Robin Lamarche-Perrin. LPAggreg is used by Ocelotl to perform the trace aggregation.  
**[https://github.com/dosimont/lpaggreg](https://github.com/dosimont/lpaggreg)**  
  
A JNI wrapper is also available to use this library with java.  
**[https://github.com/dosimont/lpaggregjni](https://github.com/dosimont/lpaggregjni)**

## [Framesoc](http://soctrace-inria.github.io/framesoc/)

### Contributor

Framesoc is the core software infrastructure of the SoC-Trace project. It provides a graphical user environment for execution-trace analysis, featuring interactive analysis views as Gantt charts or statistics views. It provides also a software library to store generic trace data, play with them, and build other analysis tools.  

![Framesoc](/images/framesoc.png)

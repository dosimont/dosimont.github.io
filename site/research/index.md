---
layout: page
title: Research
tags: []
modified: 2014-08-08T20:53:07.573882-04:00
comments: true
share: false
image:
  feature: sample-image-1.jpg
---

My research is mainly focused on the problematic of trace visualization of huge execution traces.
Trace visualization techniques are commonly used by developers to understand, debug,
and optimize their applications. Most of the analysis tools contain spatiotemporal representations, which enable to link the dynamic of the application to its structure
or its topology. However, they suffer from scalability issues and are incapable of providing
overviews for the analysis of huge traces that have at least several Gigabytes and contain
over a million of events. This is caused by screen size constraints, performance that is
required for a efficient interaction, and analyst perceptive and cognitive limitations. Indeed, overviews are necessary to provide an entry point to the analysis, as recommendedby Shneiderman’s mantra — Overview first, zoom and filter, then details-on-demand —,
a guideline that helps to design a visual analysis method.

To face this situation, I have elaborated several scalable analysis methods
based on visualization. They represent the application behavior both over the temporal
and spatiotemporal dimensions, and integrate all the steps of Shneiderman’s mantra, in
particular by providing the analyst with a synthetic view of the trace. These methods
are based on an aggregation method that reduces the representation complexity while
keeping the maximum amount of information. As this technique shows the behavior heterogeneity in the trace, 
it helps to find anomalies in embedded multimedia applications and in parallel applications running
on a computing grid, in particular.

I have implemented these techniques into Ocelotl, an analysis tool developed during
my doctorate thesis. I designed it to be capable to analyze traces containing up to several billions
of events. Ocelotl also proposes effective interactions to fit with a top-down analysis
strategy, like synchronizing our aggregated view with more detailed representations, in
order to find the sources of the anomalies.

### Main Publications

- **A Spatiotemporal Data Aggregation Technique for Performance Analysis of Large-scale Execution Traces**. Damien Dosimont, Robin Lamarche-Perrin, Lucas Mello Schorr, Guillaume Huard, Jean-Marc Vincent, *IEEE Cluster 2014*, pages 149-157, Madrid, September 2014. [![DOI](/images/doi.png)](http://dx.doi.org/10.1109/CLUSTER.2014.6968741) [![PDF](/images/pdf.png)](https://hal.inria.fr/hal-01065093/document)
- **Efficient Analysis Methodology For Huge Application Traces**. Damien Dosimont, Generoso Pagano, Guillaume Huard, Vania Marangozova-Martin, Jean-Marc Vincent, *2014 International Conference on High Performance Computing & Simulation*, pages 951-958, Bologna, June 2014. [![DOI](/images/doi.png)](http://dx.doi.org/10.1109/HPCSim.2014.6903791) [![PDF](/images/pdf.png)](https://hal.inria.fr/hal-01065783/document)

### Research Projects

- [List of projects](/site/projects/)

### Publications and Talks

- [List of publications](/site/publications/)
- [List of talks](/site/talks/)

### Main Collaborations



[Guillaume Huard](http://www-id.imag.fr/Laboratoire/Membres/Huard_Guillaume/public_html/index.html), [Jean-Marc Vincent](http://mescal.imag.fr/membres/jean-marc.vincent/index.html/), [Robin Lamarche-Perrin](http://www.mis.mpg.de/jjost/members/robin-lamarche-perrin.html), [Lucas Mello Schnorr](http://www.inf.ufrgs.br/~schnorr/), [Generoso Pagano](http://mescal.imag.fr/membres/generoso.pagano/), Youenn Corre, [Arnaud Legrand](http://mescal.imag.fr/membres/arnaud.legrand/), [Vania Marangozova-Martin](http://nanosim.imag.fr/membres/vania.marangozova-martin/), Damien Rousseau, [David Beniamine](http://moais.imag.fr/membres/david.beniamine/index.en.html)

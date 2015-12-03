---
layout: page
title: Diary
tags: []
comments: true
image:
  feature: sample-image-1.jpg
---

##Thu Dec  3 12:31:53 CET 2015

### Some propositions of short term tasks

#### Compare the result obtained with BSC Clustering-based analysis flow with Ocelotl

- [ ] Get one or several use cases: Paraver traces that are described in BSC papers. It is important to get information about the context, the behavior, and what the analysis should highlight. These traces must contain the MPI events that will be converted as punctual events, states and communication, as well as the information collected by Extrae related to the different counters used to perform the clustering.
- [ ] Step 1: Reproduce, for each trace, the analysis flow that has been performed in the papers (instrumentation step and trace collect are obsviously not required). TODO: detail this step.
- [ ] Step 2: Analysis based on classic metrics (MPI events)
  - [ ] Convert the trace to a format known by Framesoc, for instance using the converting tool developed by Youenn that generates a pjdump trace. If necessary, do the required adjustment to make the program compliant with the trace semantics.
  - [ ] Import the trace in Framesoc. Check its integrity.
  - [ ] Observe its Gantt chart. Note the differences and the similarities with Paraver Gantt chart.
  - [ ] Analyze it using Ocelotl and the classic metrics (MPI events). Compare the phases that are obtained with the behavior highlighted in BSC papers.
- [ ] Step 3: Analysis based on the counter metrics.
  - [ ] The starting point here is the Paraver trace that contains the counter metrics. We have to possibilities:
    - [ ] Take as an input the file containing the counter values associated with each CPU burst, which are dimensionnaly reduced and filtered. In this case, it is necessary to make an importer to Framesoc/Ocelotl.
    - [ ] Take as an input the file containing the counter values associated with each CPU burst, without filtering nor dimeensionnal reduction. This can enable to measure the impact of this operation on clustering, for instance.


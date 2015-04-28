---
layout: post
title: "Ph.D Thesis Defense"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2015-04-28T04:03:01+02:00
---
My Ph.D thesis defense will take place on Wednesday the 10th of June 2015, 13:30 PM (UTC+2)  
in the "Grand Amphithéâtre" of  
Inria Grenoble Rhône Alpes  
655 Avenue de l'Europe  
38330 Montbonnot-Saint-Martin  
France
  
    
      

<iframe src="https://www.google.com/maps/embed?pb=!1m14!1m8!1m3!1d5621.147739983848!2d5.807397938471249!3d45.2159579125406!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x0%3A0x60b3d32a7c64c8f0!2sInria+Grenoble+Rh%C3%B4ne-Alpes!5e0!3m2!1sfr!2sfr!4v1430126613768" width="400" height="300" frameborder="0" style="border:0"></iframe>
  

### Abstract

Trace visualization techniques are commonly used by developers to understand, debug, and optimize their applications.
Most of the analysis tools contain spatiotemporal representations, which is composed of a time line and the resources involved in the application execution. These techniques enable to link the dynamic of the application to its structure or its topology.
However, they suffer from scalability issues and are incapable of providing overviews for the analysis of huge traces that have at least several Gigabytes and contain over a million of events. This is caused by screen size constraints, performance that is required for a efficient interaction, and analyst perceptive and cognitive limitations. Indeed, overviews are necessary to provide an entry point to the analysis, as recommended by Shneiderman's \emph{mantra} - Overview first, zoom and filter, then details-on-demand -, a guideline that helps to design a visual analysis method.

To face this situation, we elaborate in this thesis several scalable analysis methods based on visualization. They represent the application behavior both over the temporal and spatiotemporal dimensions, and integrate all the steps of Shneiderman's mantra, in particular by providing the analyst with a synthetic view of the trace.
These methods are based on an aggregation method that reduces the representation complexity while keeping the maximum amount of  information. Both measures are expressed using information theory measures. We determine which parts of the system to aggregate by satisfying a trade-off between these measures; their respective weights are adjusted by the user in order to choose a level of details. Solving this trade off enables to show the behavioral heterogeneity of the entities that compose the analyzed system. This helps to find anomalies in embedded multimedia applications and in parallel applications running on a computing grid.

We have implemented these techniques into Ocelotl, an analysis tool developed during this thesis. We designed it to be capable to analyze traces containing up to several billions of events. Ocelotl also proposes effective interactions to fit with a top-down analysis strategy, like synchronizing our aggregated view with more detailed representations, in order to find the sources of the anomalies.

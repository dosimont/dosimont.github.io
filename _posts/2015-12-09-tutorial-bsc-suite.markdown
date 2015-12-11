---
layout: post
title: "Taking in Hands BSC Performance Tools Suite"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2015-12-09T17:41:19+01:00
---

This post summarize the difference step to take in hands the tool designed by the BSC Performance Tools Team, and whose purpose is to enable the analysis of complex parallel programs.

In this tutorial, we'll execute applications on MareNostrum, the supercomputer hosted by BSC. We'll focus on well-known parallel applications, like the NASPB BT application.

## Compile the BT application

First, download the NAS Parallel Benchmark. We'll use the version [3.3.1](http://www.nas.nasa.gov/assets/npb/NPB3.3.1.tar.gz).
Copy the archive on your MareNostrum account.  
`scp NPB3.3.1.tar.gz {user}@dt01.bsc.es:~`





NASPB
  `cd config`
`mv make.def.template make.def`
replace f77 by mpif77

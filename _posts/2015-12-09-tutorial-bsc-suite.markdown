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

This post summarizes the different steps to take in hands the tools designed by the BSC Performance Tools Team, and whose purpose is the analysis of complex programs running on parallel systems. In this tutorial, we'll execute applications on MareNostrum, the supercomputer hosted by BSC. We'll focus on well-known parallel applications, like the NASPB BT application.

## Compile the BT application

First, download the NAS Parallel Benchmark. We'll use the version [3.3.1](http://www.nas.nasa.gov/assets/npb/NPB3.3.1.tar.gz).
Copy the archive on your MareNostrum account.

    cp NPB3.3.1.tar.gz {user}@dt01.bsc.es:~

Connect to your account.

    ssh {user}@mn1.bsc.es

Extract the files from the archive.

    tar -xzf NP3.3.1.tar.gz

We'll work with the MPI version.

    cd NPB3.3.1/NPB3.3-MPI/

We have to set the good compiler for fortran in NASPB configuration files.

    cd config
    cp make.def.template make.def

Open make.def with your favorite editor, and change `MPIF77 = f77` by `MPIF77 = mpif77`

Now, let's compile BT, for 4 processes, class A.

    cd ..
    make BT NPROCS=4 CLASS=A






NASPB
  `cd config`
`mv make.def.template make.def`
replace f77 by mpif77

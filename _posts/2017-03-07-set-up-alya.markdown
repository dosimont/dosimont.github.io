---
layout: post
title: "Set Up Alya"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2017-03-07T16:38:00+02:00
---

### Instructions to install Alya on a Fedora 25 distribution

Dependencies:

 `sudo dnf install make automake gcc gcc-c++ kernel-devel`
 `sudo dnf groupinstall "Development Tools" "Development Libraries"`
 `sudo dnf install gcc-gfortran`
 `sudo dnf openmpi openmpi-devel`

Download the svn repository (requires an access to BSC's network):

 `svn co svn+ssh://[user]@dt01.bsc.es/gpfs/projects/bsc21/svnroot/Alya/Trunk/`



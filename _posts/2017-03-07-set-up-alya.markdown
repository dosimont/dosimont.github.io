---
layout: post
title: "Set Up Alya"
modified: 2017-03-17
categories: 
excerpt:
tags: []
image:
  feature:
date: 2017-03-07
---

## Dependencies

    sudo dnf install make automake gcc gcc-c++ kernel-devel 
    sudo dnf groupinstall "Development Tools" "Development Libraries"  
    sudo dnf install gcc-gfortran
    sudo dnf openmpi openmpi-devel  

You may have to add the path of your mpi binaries to your `PATH`

## Download the svn repository

Download the svn repository (requires an access to BSC's network):

    svn co svn+ssh://[user]@dt01.bsc.es/gpfs/projects/bsc21/svnroot/Alya/

 Using `git+svn`:

    git svn clone svn+ssh://[user]@dt01.bsc.es/gpfs/projects/bsc21/svnroot/Alya/


## Compile Alya

    cd Trunk/Executables/unix

Copy the configuration file related to your compiler/platform. Ex: compiling using gfortran:

    cp configure.in/config_gfort.in config.in"

Compile

    ./configure -x turbul nastin parall
    make metis4
    make -j[number of processor]





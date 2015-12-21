---
layout: post
title: "Taking in Hands the BSC Performance Tools Suite"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2015-12-09T17:41:19+01:00
---

This post summarizes the different steps to take in hands the tools designed by the BSC Performance Tools Team, and whose purpose is the analysis of complex programs running on parallel systems. In this tutorial, we'll execute applications on MareNostrum, the supercomputer hosted by BSC. We'll focus on well-known parallel applications, like the NASPB BT application.

## Install the BSC Performance tools

First, I recommand to install the BSC Performance tools, from the [website](https://www.bsc.es/computer-sciences/performance-tools/downloads).

### Paraver

Paraver is a tool which provides several ways to analyze a trace. The main visualization is a space-time representation, showing the application behavior over time, for each process/task/resource.
There is also an interesting mechanism, which consists in loading *configuration files*, in order to build a representation (spatiotemporal, statistics, table, etc.) based on a particular aspect or metrics (for instance MPI events, communications, clustering, etc.). Last but not least, the tool enables to filter trace events, cut time periods, and perform other operations that can help to reduce the trace size to facilitate the analysis.

I strongly advise to install Paraver using the binaries, since compiling from the sources may be tedious.
In this tutorial, we assume you put the Paraver binary directory in your PATH, in order to launch paraver from anywhere using the command line.

*The Paraver version used in this tutorial is 4.5.8.*

### Extrae

Extrae is a tool used to trace the application and generate a paraver trace file.
In this tutorial, we don't install Extrae on our personal computer, since a version is already present on MareNostrum, which we use to run our experiments and trace our applications.
However, if you don't have an access to MareNostrum, you may want to install Extrae on your computer (or on a computing grid). Please consult the [documentation] for further information.

*Careful: I had to face one bug (related to the trace merging process), which required the utilization of a development version and not the version provided on MareNostrum.*

### Dimemas

For the moment, we don't focus on Dimemas, which is a parallel system simulator.

### Clustering Suite

The Clustering Suite is mainly [Juan Gonzalez](https://www.researchgate.net/profile/Juan_Gonzalez27)'s work. It serves as a base for several analysis. Its principle consists in clustering application CPU bursts (which correspond to the period where the application is computing, just before a communication), in order to distinguish the parts of the application that are the most costly and/or the less efficient.

You can install it from the sources of from the binary. Don't forget to put the binary directory in your PATH.

*The Clustering version used in this tutorial is 2.6.4.*

### Folding

Folding is [Harald Servat](https://www.bsc.es/es/about-bsc/staff-directory/servat-gelabert-harald)'s work. It enables to compute performance metrics from repetitive delimited regions, thanks to a sampling mechanism used during the tracing, and the clustering result.

*We use, in this tutorial, a development version.*


## Compile the BT application

Now, let's try to analyze an application using these tools. We'll use the NAS Parallel Benchmark, which provide applications whose behavior is easily understandable. First, download it. We'll use the version [3.3.1](http://www.nas.nasa.gov/assets/npb/NPB3.3.1.tar.gz).
Copy the archive on your MareNostrum account.

    $ cp NPB3.3.1.tar.gz {user}@dt01.bsc.es:~

Connect to your account.

    $ ssh {user}@mn1.bsc.es

Extract the files from the archive.

    @ tar -xzf NP3.3.1.tar.gz

We'll work with the MPI version.

    @ cd NPB3.3.1/NPB3.3-MPI/

We have to set the good compiler for fortran in NASPB configuration files.

    @ cd config
    @ cp make.def.template make.def

Open make.def with your favorite editor, and change `MPIF77 = f77` by `MPIF77 = mpif77`

Now, let's compile BT, let's say for 64 processes, class B.

    @ cd ..
    @ make bt NPROCS=64 CLASS=B

## Configuring the job

Once we have compiled the program, we should copy the binary into the directory that will be used to launch the job.
Indeed, on Marenostrum, we are strongly advised to launch jobs from the *scratch* or *group* directory, because of the file system configuration (see [Marenostrum documentation](https://www.bsc.es/support/MareNostrum3-ug.pdf))

    @ mkdir ~/scratch/experiment
    @ cp bin/BT.B.64 ~/scratch/experiment/

Here, I use an alias to access to *scratch*. I advise to create symbolic links using this way, for the *scratch* and the *group* directory. I will keep this convention all the way along.

Now, we have to set the path to Extrae, which will be used to trace the application. Let's add this in your _.bashrc_ file.

    @ cd EXTRAE_HOME=/apps/CEPBATOOLS/extrae/latest/default/64

Now, let's create the scripts which will be used to launch the job.

    @ cd ~/scratch/experiment/
    @ touch job.sh trace.sh
    @ chmod 755 job.sh trace.sh

Let's describe briefly what we should put in these scripts:

### job.sh

This is the job script submitted to the job scheduler through the command

    bsub < job.sh

    #!/bin/bash
    #BSUB -n 64                        //The number of processes 
    #BSUB -o output_%J.out             //The output file
    #BSUB -e output_%J.err             //The error file
    #BSUB -R "span[ptile=16]"          //The value ptile corresponds to the number of MPI processes on each node
    #BSUB -J mpi_extrae_test           //The job name
    #BSUB -W 01:00                     //The job duration (HH:MM)

    APP=bt.B.64                        //The binary file

    module load extrae                 //Load the module extrae
    mpirun ./trace.sh $APP             //Run the application using a wrapper, the file trace.sh

### trace.sh

This file is a wrapper that is used to enable the application tracing, using the library dynamic preload feature provided by Extrae (this method is stabler than the DynInst feature).

    #!/bin/sh
    export EXTRAE_CONFIG_FILE=./extrae.xml
    export LD_PRELOAD=${EXTRAE_HOME}/lib/libmpitracef.so
    ./$*

## Configuring the tracing

Once we have written the scripts (and adapted the parameters if required), we have to configure Extrae through its configuration file, extrae.xml.
First, copy this file from the example directory of Extrae:

    @ cp ${EXTRAE_HOME}/share/example/MPI/extrae.xml .

Take care that the CPU bursts tracing, which is necessary to perfom the clustering, is activated in extrae.xml:

    <bursts enabled="yes">
      <threshold enabled="yes">500u</threshold>
      <mpi-statistics enabled="yes" />
    </bursts>

It is also important to check that the trace merging step is done automatically:

    <merge enabled="yes" 
      synchronization="default"
      tree-fan-out="16"
      max-memory="512"
      joint-states="yes"
      keep-mpits="yes"
      sort-addresses="yes"
      overwrite="yes"
    />

## Submit the job

Now, submit the job, using the following command:

    @ bsub < job.sh

and wait for the job termination.

## Execution result

You now should have in your directory the application outputs (standard and error). You can take a look to check that everything was ok.
Paraver trace files (.prv, .pcf, .row) should also be present. If it's not the case, investigate: something wrong probably happened during the application execution.

## Analyze the trace using the clustering technique

Let's download the three trace files on our personal computer. Let's say that they are now contained in an *analysis* directory.

    $ cd analysis

First, you should get an example of the clustering configuration file, provided in its share directory

    $ cp ${CLUSTERING_HOME}/share/example/cluster.xml .

If we applied direcly the clustering without set the parameters, we would have to deal with too many cpu bursts. Some of them are not interesting: they are too short and don't represent a significant part in the computation. As our main objective is to focus on parts of the application we can improve, they are not relevant to us. To discard these cpu bursts, we use the stat tool.

    $ stats bt.B.64.prv -bursts_histo

This generates a plot `bt.B.64.bursts.gnuplot`, containing a graph, showing the number of bursts as a function of their duration, and an histogram, showing the percentage of running time as a function of the duration. Let's open it using this command:

    $ gnuplot -p bt.B.64.bursts.gnuplot

![](/bursts.png)

The idea is to select the duration, below which most of the bursts are contained, but whose total running time is not significant, to discard the maximum of cpu bursts without hinder the analysis.
In the illustration, whe choose 10 us.
We put this value in the cluster.xml file, in the `duration_filter` field:

    <clustering_definition
    use_duration="no"
    apply_log="yes"
    normalize_data="yes"
    duration_filter="10"
    threshold_filter="0">

Now, let's have an overview of the point that are discarded:

    $ ClusteringDataExtractor -d cluster.xml -i bt.B.64.prv
    $ gnuplot -p bt.B.64.IPC.PAPI_TOT_INS.gnuplot

![](/ipc_without_range.png)

On this picture, we can see that we discarded many points, in black, but there is still many points left, whose coordinates are close to black ones: some of them could be removed to facilitate the analysis.
By zooming, we can see several clusters of points. We can consider to discard clusters whose total instruction number is low, since the IPC number which is associated is also weak.
To do this, we modify the cluster.xml file again, and filter the total instruction number by discarding points that are less than 8e5 instructions:

    <single_event apply_log="yes" name="PAPI_TOT_INS">
    <event_type>42000050</event_type>
    <range_min>8e5</range_min> 
    </single_event>

![](/ipc_range.png)

Now, we want to set espilon and the min points values, that are required to perform the clustering algorithm:

    <clustering_algorithm name="DBSCAN">
    <epsilon>.001</epsilon>
    <min_points>10</min_points>
    </clustering_algorithm>

A value of 10 points is recommended, but we still have to tune espilon. Basically, a higher value of epsilon gives big clusters, but requires a long computation time. On the contrary, a small value of epsilon provides small clusters, and necessitates a short computation time. The objective is to find relevant clusters, by grouping only points that are in the same area. This necessitates to try different value of epsilon to find a good trade-off.

    $ BurstClustering -d cluster.xml -i bt.B.64.prv -o bt.B.64.clustered.prv

## Shneiderman's mantra is never far...

You may see that the computation time required to generate the clusters is long, even with a small espilon: in my case, the trace I generated was 1.6 GB. Even by removing non relevant points, the number of remaining points is too high for the clustering algorithm. It is thus necessary to reduce the trace size.
To perform this operation, we propose to use Paraver.
First, open the trace:

    $ wxparaver bt.B.64.prv







    





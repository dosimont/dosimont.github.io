---
layout: post
title: "Using Alya's parallel I-O"
modified: 2017-06-07
categories: 
excerpt:
tags: [Alya]
image:
  feature:
date: 2017-05-03
---

We describe on this page how to use the parallel I/O feature newly implemented into Alya.

## Compilation

Beside enabling MPI, nothing in particular is required to compile Alya with the parallel I/O feature.

### Classic configuration

Follow the classic configuration procedure:

Go into the compilation directory:

    $ cd Executables/unix

Edit `config.in` according to your needs and/or copy it from `configure.in`.

Configure. For instance to compile Alya with nastin:

    $ ./configure -x parall nastin

Be sure to remove existing object files:

    $ make clean

Compile metis:

    $ make metis4

Compile:

    $ make -j4
  
> Note that the parallel I/O feature is currently incompatible with the -DI8 compiler option.

### Benchmarking parallel I/O

You may want to record some statistics about the time passed in each I/O operation.
In this case, you have to add the following line to the `config.in` file before executing `configure`:

    CSALYA   := $(CSALYA) -DBENCHMARK

## Convert your mesh file

The parallel I/O feature requires a [specific file format](/binary-format).
This format is used for the mesh description as well as for the fields (input, post process or restart files).
Consequently, it is necessary to convert the ascii input file describing the mesh into this new format.

You can use the Alya to MPI-IO tool available [here](https://github.com/dosimont/alya-mpio-tools). Compile it as follow:

    $ git clone https://github.com/dosimont/alya-mpio-tools.git
    $ cd alya-mpio-tools
    $ mkdir -p build
    $ cd build
    $ cmake ..
    $ make

Eventually, you may want to install it:

    $ sudo make install

Refer to CMake's documentation to change the installation path or the compilation options...

Once you have installed the tool, convert your mesh file as follow:

    $ mpio-alya -e [n] -b [m] -n [case-name] [geometry-file]

The options `-e`, `-b` are not mandatory but strongly advised to optimize the resulting binary file.
`-e n` defines the maximum number of points `n` that an element contains in the mesh file.
`-b m` defines the maximum number of points `m` that a boundary contains in the mesh file.
Both options help to correctly align the binary file records, which is essential to allow an efficient parallel reading.

`-n case-name` defines the string used to name the output files.

`geometry-file` is the file containing the geometry data. Be aware that you have to specify the file that actually contains the data; the includes are not considered by the tool.

Running the tool will generate several files with the extension `.mpio.bin`.
Note that the tool only considers the following fields:

  - `NODES_PER_ELEMENT`
  - `TYPES`
  - `ELEMENTS`
  - `BOUNDARIES`
  - `COORDINATES`

The remaining data is exported to the file `case-name.mpio.geo`. **In the domain file `.dom.dat`, you must replace the include of the original `geometry-file` by an include of this file.**

Example:

    GEOMETRY
      INCLUDE ./geometry-file
    END_GEOMETRY

Becomes:

    GEOMETRY
      INCLUDE ./case-name.mpio.geo
    END_GEOMETRY
    
## Configure the `.dat` file

In order to enable the parallel I/O feature during Alya's execution, you have to modify your main `.dat` file.
Here, an example of the `PARALL_SERVICE` section:

    PARALL_SERVICE:         On
      IO:                   On
        READING:            On
        REORDERING:         On
        RESTART:            On
        COMMUNICATOR:       SFC
      END_IO
      PARTITION:            FACES
      PARTITIONING:
        METHOD:              SFC
      END_PARTITIONING
    END_PARALL_SERVICE

### IO section

To activate the parallel I/O, you need to define the `IO` section inside the `PARALLEL_SERVICE` section

      IO:                   On
        READING:            On
        REORDERING:         On
        RESTART:            On
        COMMUNICATOR:       SFC
      END_IO

The field `READING` enables (`ON`) or disables (`OFF`) the parallel reading of the mesh file.
The field `REORDERING` enables (`ON`) or disables (`OFF`) the parallel writing of a renumbered mesh.
This renumbered mesh is generated after the partition.
The field `RESTART` enables (`ON`) or disables (`OFF`) the parallel writing and reading of the postprocess and restart files.
Note that this option will erase the post/restart format specification, since only the MPIO binary format is compatible with the parallel I/O.
The field `COMMUNICATOR` specifies the strategy to define the communicator responsible for the input mesh file reading.
You can choose between `SFC` and `MODULO n` with n an integer from 1.
If the partitioning method defined in the section `PARTITIONING` is `SFC`, it is strongly advised to select `SFC` in the `IO` section `COMMUNICATOR` field too.
Note that the `SFC` strategy is compatible with any of the partitioning techniques (METIS, etc.).
The `MODULO n` technique defines a communicator of m processes such that m=1+(nprocs-1)/n.
Note that the writing and reading of the postprocess and restart files use the default Alya's communicator, involving all the processes. Currently, this cannot be modified.

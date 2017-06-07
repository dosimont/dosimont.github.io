---
layout: post
title: "Alya MPI-IO configuration"
modified: 2017-05-03
categories: 
excerpt:
tags: [Alya]
image:
  feature:
date: 2017-06-07
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

## Convert your mesh files

The parallel I/O feature requires a specific file format
This format is used for the mesh description as well as for the field files (in input, post process or restart).
Consequently, it is necessary to convert the ascii file format commonly used by Alya into this new format.

You can use the Alya to MPI-IO tool available [https://github.com/dosimont/alya-mpio-tools](here). Compile it as follow:

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
`-e n` defines the maximum point number `n` that an element contains in the mesh file.
`-b m` defines the maximum point number `m` that a boundary contains in the mesh file.
Both options help to align correctly the binary file records, which is essential to allow an efficient parallel reading.

`-n case-name` defines the string used to name the output files.

`geometry-file` is the file containing the geometry data. Be aware that you have to specify the file that actually contains the data; the includes are not considered by the tool.

Running the tool will generate several files with the extension `.mpio.bin`.
Note that the tool only considers the following fields:

    - `NODES_PER_ELEMENT`
    - `TYPES`
    - `ELEMENTS`
    - `BOUNDARIES`
    - `COORDINATES`

The remaining data is exported to the file `case-name.mpio.geo`. 

**In the domain file `.dom.dat`, you must replace the include of the original `geometry-file` by an include of this file.**

Example:

    GEOMETRY
      GROUPS = 2000
      INCLUDE ./geometry-file
    END_GEOMETRY

Becomes:

    GEOMETRY
      GROUPS = 2000
      INCLUDE ./case-name.mpio.geo
    END_GEOMETRY
    
## Configure

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

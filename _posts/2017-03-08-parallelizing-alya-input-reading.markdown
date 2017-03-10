---
layout: post
title: "Parallelizing Alya's Input Reading"
modified: 2017-03-10
categories: 
excerpt:
tags: [Alya]
image:
  feature:
date: 2017-03-08
---

This page summarizes the information and the discussions regarding the parallelization of Alya's input files reading.

# Alya's input mesh file

Input mesh files describing unstructured meshes, of extension `.dat.geo`, are single textual files composed of several sections.

Section `TYPES` assignes a type (a geometrical figure, e.g. a triangle, a rectangle,...) to each geometrical element.
The first column is the mesh identifier and the second one the type identifier ; both are unsigned integers. The value of those types is defined in Alya.

    TYPES  
    1 37  
    2 37  
    3 34  
    ...  
    END_TYPES


Section `COORDINATES` defines the coordinate of each node that will be use to describe the geometrical figures. A same node can be shared by several elements.

    COORDINATES  
    1   0.0000000000000e+00  0.0000000000000e+00  0.0000000000000e+00  
    2  -1.1838919219742e-03  1.9714187252377e+00  0.0000000000000e+00  
    3   2.0000000000000e+00  0.0000000000000e+00  0.0000000000000e+00  
    ...
    END_COORDINATES

Section `ELEMENTS` defines the elements, or meshes, themselves. The first index is the element identifier, and the following unsigned integer numbers corresponds to the nodes that are located on the apexes or sides of the elements. Their order matters. It is noticeable that the node number is not regular and may differ at each line.

    ELEMENTS  
    1 9 1 10 11 2372 2410 1384 1385  
    2 10 12 13 11 1384 1386 1387 1385  
    3 11 14 15 9 1385 1388 1002 2372  
    ...  
    END_ELEMENTS

Section `BOUNDARIES`

    BOUNDARIES  
    1 9 1 10 11  
    2 10 12 13 11  
    3 11 14 15 9  
    ...  
    END_BOUNDARIES

_Remark: 
merging the types and the elements sections would help to optimize the parsing and reduce the trace size: types could be, for instance, the second index of the elements section._

_Edit: by looking at Alya's code, it appears that addtional sections can appear in the input file_

# Objective

Our objective is basically to reduce the reading time as well as minimizing the memory footprint of this operation. Parallelizing the I/O using a distributed file system and a parallel I/O library could help to do so.

# Constraints

1. Gain performance
2. Minimize the memory footprint
3. Avoid external dependencies as much as possible
4. Keep the input format for parallel reading identical or close to the original input format

# Parallelizing with MPI-IO

Advantages:
- efficient
- no external API
- stable
- reading and writing are very similar to MPI send and receive

Drawbacks:
- is not really compliant with textual files and works much better with binary files
  - does not read ASCII and is thus unable to detect line breaks
- parsing files containing several sections or having an irregular structure may be troublesome

### Issues

Basically, to improve performance and be compliant with MPI-IO, it is mandatory to specify a convenient trace format whose structure and content suits to its parallel reading.
This is not the case of the current format.

#### Textual input

MPI-IO does not read ASCII, so reading a textual file would add some processing.
Parsing the different sections would require to know their location in the file to be efficient.  
The easiest solution is split the input file: one file per section.
  - Easy to pass from the single file to the split files with a simple perl script (however, this solution does not suit to huge files because of the computation time/memory overload)
  - Requires to modify the workflow to generate such input files and read them
  - Retrocompatibility is not ensured  

Since each line in the `ELEMENTS` section may be of a different length, manage the irregular lines can be tricky, in particular when we partition the file to feed the processes.  
A potential solution is overlapping. The technique has been described [here](http://stackoverflow.com/questions/13327127/mpi-io-reading-file-by-processes-equally-by-line-not-by-chunk-size) and [here](http://stackoverflow.com/questions/12939279/mpi-reading-from-a-text-file) for cases that are very similar to ours.

#### Binary input

Using a binary file would be much more compliant with MPI-IO. Also, reading a binary file is faster, even in sequential. However, the file is not human-readable anymore. Other issues may be related to endianness.
See [here](https://www.sharcnet.ca/help/index.php/Parallel_I/O_introductory_tutorial#Data_Formats) for more info.

Of course, switching to a binary format with new specifications does not fulfill the constraint **4**.

Some guidelines that could help to specify this format in order to ensure its compliancy with MPI-IO

- Header expressing the file structure:
  - the item/line number of each section
  - sections must be ordered and well defined
- Body:
  - each item must be aligned
  - It would be convenient to use the same format whatever the section to help the displacement computation
    - This could be done by using two dimension arrays and replicating the elements when it's required

_In binary_

    ELEMENTS  
    1 9 
    1 1 
    1 10 
    1 11 
    ...
    2 10
    ...
    END_ELEMENTS

This [page](https://en.wikipedia.org/wiki/STL_(file_format)) offers a good example of a binary format through the STL format specification.

# Analysis of Code-Saturn

Ricard is in contact with someone working on Code-Saturn, an open source software solving Navier-Stokes equations for several flows, available [here](http://code-saturne.org/cms/).
They developed a home-made solution for parallel I-O. Let's take a look on this to get some inspiration.

## Input files

The input files provided in the example directory of Code-Saturn are binaries. I didn't find more information about the format (.des), which seems to be home-made.

## Code

Charles, Ricard's contact, confirmed that both reading and writing can be done in parallel (or serial if MPI-IO is not present on the machine).
The directories and files we should look for:

    src
    |
    |------ base
            |
            |------ cs_base.*
            |------ cs_io.*
            |------ cs_file.*
    |        
    |------ fvm
            |
            |------ *


### Base directory

#### `cs_base`

This file contains some base functions for manipulating string, files, i/o, etc.

#### `cs_io`

This file is probably the most interesting since it contains functions to read/write headers, sections, blocks.
Interesting content of `cs_io.h` about the concept of section (does it correspond to _our_ sections, as we have defined above?).

    /*----------------------------------------------------------------------------
     * Write a global section.
     *
     * Under MPI, data is only written by the associated communicator's root
     * rank. The section data on other ranks is ignored, though the file offset
     * is updated (i.e. the call to this function is collective).
     *
     * parameters:
     *   section_name     <-- section name
     *   n_vals           <-- total number of values
     *   location_id      <-- id of associated location, or 0
     *   index_id         <-- id of associated index, or 0
     *   n_location_vals  <-- number of values per location
     *   elt_type         <-- element type
     *   elts             <-- pointer to element data
     *   outp             <-> output kernel IO structure
     *----------------------------------------------------------------------------*/

#### `cs_file`

This file is responsible, inter alia, of the MPI-IO calls.

### FVM directory

TO BE COMPLETED

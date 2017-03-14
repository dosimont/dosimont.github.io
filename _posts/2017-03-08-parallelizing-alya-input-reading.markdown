---
layout: post
title: "Parallelizing Alya's Input Reading"
modified: 2017-03-14
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

_Edit 2017-03-12: the information presented in this section is not complete, since the file that have been provided to me is not similar to the file format described in the_ [documentation](http://bsccase02.bsc.es/alya/Domain_input_data.html). _Refer to this documentation rather than to what follows._

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

Of course, switching to a binary format with new specifications does not fulfill the constraint **4**. However, after a code analysis of Alya, it appears that the parser is able to read a certain binary format.

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

or by defining an arbitrary number of elements per row. This information should be present in the header/metadata file.

This [page](https://en.wikipedia.org/wiki/STL_(file_format)) offers a good example of a binary format through the STL format specification.

Another possibility is to use an already-existing description format, such as the ones used by [ParaView](http://www.paraview.org/Wiki/ParaView/Data_formats). Basically, it seems that such formats are composed of a binary, which contains the data, and a separate xml file containing the metadata. An exemple following this structure is [VTK](http://www.cacr.caltech.edu/~slombey/asci/vtk/vtk_formats.simple.html). Thes question is: are one of these file formats able to represent all the data required by Alya? If yes, the idea would be to follow one of the popular _standard_ format.
Some [fortran libraries](http://xml-fortran.sourceforge.net/) already exist to parse/generate xml, which could be convenient to deal with the metadata.

This page explains the [parallel pipeline](http://www.vtk.org/Wiki/VTK/Parallel_Pipeline) used by VTK.
Another link for [parallel reading of mesh files](http://www.paraview.org/ParaView/index.php/Parallel_I/O#Optimized_Parallel_Readers).

# Parallel Workflow

We design here the parallel workflow.

## Generating the binary file

First, it's necessary to generate a file format that suits to MPI-IO. Let's show the flow for a binary format with xml metadata.
Here, it's necessary to design a program (`Binary Converter`) that converts from the original input format to the binary format. Note that this step is repeated once, but can be time costly and limited by the available memory.

    Mesh Designer Program -> file.dom.dat -> Binary Converter -> file.{bin} + file.{xml}

The better solution is to generate directly the binary format from the program used to generate the mesh description.

    Mesh Designer Program -> file.{bin} + file.{xml}

I need to get more insight about the programs that are used to describe the mesh. Maybe, the export could be realized through a plug-in or an external script integrated to the Mesh Designer Program.

## From the reading to the partitioning

### Partition with METIS

This is the current working state of Alya.

    Sequential Reading (current) -> Sequential Partitioning (METIS) (current)

Since the partition part is sequential, we are not interesting in reading in parallel for this partition method because it will add complexity. However, it's important to ensure compatibility with METIS with the new reading implementation, i.e. ensure a sequential reading.
For the sequential reading, two possibilities:

- if MPI-IO is available on the machine, just use one worker (the master) to perform the reading process instead of N workers.
- if MPI-IO is not available, it's necessary to use a sequential solution not relying on MPI-IO (MPI-IO methods will be inavailable). The best to manage both cases easily is to use a wrapper, which calls a different method according to the presence or not of MPI-IO.


### Partition with ParMETIS  

    Sequential Reading (current) -> Parallel Partitioning (ParMETIS) (??)


This time, the partition process is done in parallel. Theoretically, it could be interesting to envisage a parallel readinig, but from what I understood, the gain using ParMETIS is not really huge. Moreover, I'm not sure that a naive block reading (each process reading contiguous blocks of same size) would suit to ParMETIS and it may require to reorganize the data before the partition, which could be costly.

### Partition with SFC

    Sequential Reading (current) -> Parallel Partitioning (SFC) (not integrated yet)

### Parallel reading

Since SFC does not care about how the elements are distributed amongst the processes, this is the partition that suits the most to our parallel reading approach relying on dividing each section in blocks and attributing an equal subset to each process, whitout taking into account the data value itself in this attribution.

    Reading ---------------------------------> Parallel Partitioning (SFC)
      |                                         |
    Process 1 (Master) ----------------------> Items[1->NItems/N]
    Process 2                               -> Items[(Nitems+1)/N->2*NItems/N]
    ...
    Process N                               -> Items[((N-1)Items+1)/N->NItems] 
      |
      |
      |
    Each process contains NItems/N items
    Item[i] € Process i/NItems * N 
    
If this approach suits well already for the items of type elements, it is not the case for the nodes whose the elements depend on (and probably some other type of data), and that are originally not distributed amongts the processes.
Thus, it is necessary to modify the partition code to take this into account.
Several solutions:

- **All the node information is mutualized between the processes prior to the partitioning**.
  - Close to the current code in which the master distributes all the information to the process.
  - Theoretically and practically simple
  - Memory consuming
  - Once the sharing is done, all the processes have access to the data and thus do not require extra communications
  - `MPI_Allgather`
- **Only necessary nodes are shared between the processes**
  - Requires to modifiy the partition code more deeply
  - Requires preprocessing
  - It's easy to determine who owns a node according to its id
  - Memory saving
  - May lead to irregular communication patterns and decrease performance
  - We could use a `distributed array`: 
    - [MPI Distributed arrays](http://mpi-forum.org/docs/mpi-2.2/mpi22-report/node73.htm)
    - [Fortran Co-Array](http://www.polyhedron.com/web_images//intel/productbriefs/8_CAF.pdf)
    - [MPI RMA](http://wgropp.cs.illinois.edu/courses/cs598-s16/lectures/lecture34.pdf)
- **[...]**


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



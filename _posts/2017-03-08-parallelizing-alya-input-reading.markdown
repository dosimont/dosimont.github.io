---
layout: post
title: "Parallelizing Alya's Input Reading"
modified:
categories: 
excerpt:
tags: []
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

_Some remarks: 
Merging the types and the elements sections would help to optimize and reduce the trace size easily: types could be, for instance, the second index of the elements section._

# Objective

Our objective is basically to reduce the reading time as well as minimizing the memory footprint of this operation. Parallelizing the I/O using a distributed file system and a parallel I/O library could help to do so.

# Constraints

- Gain performance
- Minimize the memory footprint
- Avoid external dependencies as much as possible
- Keep the input format for parallel reading identical or close to the original input format

# Technologies

## MPI-IO

Advantages:
- no external API
- stable 

Drawbacks:
- works much better with binary files

### Issues:

Parsing the different sections would require to know their location in the file to be efficient.

Envisaged solutions:
- Split the input file: one file per section
  - Easiest solution
  - Easy to pass from the single file to the split files with a simple perl script
  - Requires to modify the workflow to generate such input files and read them
  - Retrocompatibility is not assured
- Use metadata to indicate where is positionned the beginning of each section
  - This position can not use the line number, but the file offset, which makes the things a bit tricky
  - Modifying the input file (even a single modification) will prevent it to be readen correctly
  - It is a really bad practice to mix textual data and information about the file binary content
  - This solution would be acceptable if we used a binary file

Since each line in the `ELEMENTS` section may be of a different length, manage the irregular lines can be tricky, in particular when we partition the file to be processed by several processes.
Envisaged solutions:
  - In the case of a binary file, force a regular array with a length equal to the longest row
  - In the case of a textual file, use overlapping

# Analysis of Code-Saturn

Ricard is in contact with someone working on Code-Saturn, an open source software solving Navier-Stokes equations for several flows, available [here](http://code-saturne.org/cms/).
They developed a home-made solution for parallel I-O. After a brief analysis of their code, it appears that only the writing is parallelized. I should ask for a confirmation anyway.


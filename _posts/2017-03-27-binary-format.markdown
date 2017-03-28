---
layout: post
title: "Binary Format"
modified: 2017-03-28
categories: 
excerpt:
tags: [Alya]
image:
  feature:
date: 2017-03-27
---

## Original binary format

Problem: Record-based versus Stream I/O. See [here](http://www.star.le.ac.uk/~cgp/streamIO.html)

## New binary format:

    ------------------------------------------------------------------------
    Magic number (signed int 32 bits):        27093
    --Alignment (signed int 32 bits):         0
    Format (char* 64 bits):                   MPOALYA
    Version (char* 64 bits):                  V000001
    Object (char* 64 bits):                   LNODS00/LTYPE00/LNODS00
    Type int/real (char* 64 bits):            INTEGER/REAL000
    Size (char* 64 bits):                     4BYTES0/8BYTES0
    Sorting (char* 64 bits):                  ASCENDI/DESCEND/NONE000
    Id (char* 64 bits):                       YES0000/NO00000
    --Alignment (char* 64 bits):              0000000
    Lines (signed int 32 bits):               nlines
    Columns (signed int 32 bits):             ncolumns (id not counted)
    ------------------------------------------------------------------------
    Total: 32*2+64*8+32*2 = 640 bits = 80 bytes
    
    Table:
    ------------------------------------------------------------------------
    (id:1)      i_1_1       i_1_2       ...               i_1_ncolumns
    (id:2)      i_2_1       i_2_2       ...               i_2_ncolumns
    ...
    (id:nlines) i_nlines_1  i_nlines_2  ...               i_nlines_ncolumns
    ------------------------------------------------------------------------


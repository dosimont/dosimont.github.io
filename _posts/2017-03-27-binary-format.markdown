---
layout: post
title: "Binary Format"
modified: 2017-03-27
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
    Format (char* 64 bits):                   MPIOALYA
    Version (char* 64 bits):                  V0000001
    Object (char* 64 bits):                   ELEMENTS/NODES000/COORDS00
    Type int/real (char* 64 bits):            INTEGER0/REAL0000
    Size (char* 64 bits):                     4BYTES00/8BYTES00
    Sorting (char* 64 bits):                  ASCENDIN/DESCENDI/NONE0000
    Id (char* 64 bits):                       YES00000/NO000000
    --Alignment (char* 64 bits):              00000000
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


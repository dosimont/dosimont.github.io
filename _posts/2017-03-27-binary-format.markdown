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

### Version 1:

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

### Version 2:

    ------------------------------------------------------------------------
    Magic number (signed int 32 bits):        27093
    --Alignment (signed int 32 bits):         0
    Format (char* 64 bits):                   MPIAL00 
    Version (char* 64 bits):                  V000200
    Object (char* 64 bits):                   COORD00/LTYPE00/LNODS00/LTYPB00...
    Dimension (char* 64 bits)                 SCALA00/VECTOR0
    Results on (char* 64 bits)                NELEM00/NPOIN00/NBOUN00
    Type int/real (char* 64 bits):            INTEG00/REAL000
    Size (char* 64 bits):                     4BYTE00/8BYTE00
    Seq/Parallel (char* 64 bits)              SEQUE00/PARAL00
    Filter or no filter (char* 64 bits)       FILTE00/NOFIL00
    Sorting (char* 64 bits):                  ASCEN00/DESCE00/NONE000
    Id (char* 64 bits):                       ID00000/NOID000
    --Alignment (char* 64 bits):              0000000
    Columns (signed int 32 bits):             ncolumns (id not counted)
    Lines (signed int 32 bits):               nlines (0=PARALLEL)
    Time step number (signed int 32 bits):    ittim
    Number of subdomains (signed int 32 bits):nsubd (1=SEQUENTIAL)
    Time (real 64 bits):                      time
    --Alignment (char* 64 bits):              0000000
    Option 1 (char* 64 bits):                 OPTION1 (each option is a 7+1 character, interpreted freely by the parser)
    Option 2 (char* 64 bits):                 OPTION2 (each option is a 7+1 character, interpreted freely by the parser)
    Option 3 (char* 64 bits):                 OPTION3 (each option is a 7+1 character, interpreted freely by the parser)
    Option 4 (char* 64 bits):                 OPTION4 (each option is a 7+1 character, interpreted freely by the parser)
    Option 5 (char* 64 bits):                 OPTION5 (each option is a 7+1 character, interpreted freely by the parser)
    Option 6 (char* 64 bits):                 OPTION6 (each option is a 7+1 character, interpreted freely by the parser)
    Option 7 (char* 64 bits):                 OPTION7 (each option is a 7+1 character, interpreted freely by the parser)
    Option 8 (char* 64 bits):                 OPTION8 (each option is a 7+1 character, interpreted freely by the parser)
    ------------------------------------------------------------------------
    Total: 32*2+64*12+32*4+64+64+10*64 = 1728 bits = 216 bytes
    
    Table:
    ------------------------------------------------------------------------
    (id:1)      i_1_1       i_1_2       ...               i_1_ncolumns
    (id:2)      i_2_1       i_2_2       ...               i_2_ncolumns
    ...
    (id:nlines) i_nlines_1  i_nlines_2  ...               i_nlines_ncolumns
    ------------------------------------------------------------------------


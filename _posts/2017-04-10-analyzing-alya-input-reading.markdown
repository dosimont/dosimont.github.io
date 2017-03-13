---
layout: post
title: "Analyzing Alya's Input Reading"
modified: 2017-03-10
categories: 
excerpt:
tags: [Alya]
image:
  feature:
date: 2017-03-10
---

We focus on the code from the start to the input reading.
The corresponding files are located in the following location:

    Sources
    |
    |------ kernel
            |
            |------ master
                    |
                    |------*.f90


### `Alya.f90`

This is the main file, containing the Alya program.
After some conditional initializations (Extrae, Alya DLB), call the subroutine `Turnon`.

### `Turnon.f90`

Defines the subroutine `Turnon`. Contains the following dependencies (**important**: it seems that many global variables are used, with multiple side effects!):

    use def_parame
    use def_elmtyp
    use def_master
    use def_inpout
    use def_domain
    use def_kermod
    use mod_finite_volume,        only : finite_volume_arrays

Definitions are located here:


    Sources
    |
    |------ kernel
            |
            |------ defmode
                    |
                    |------*.f90

After initializing, set up mpi communicators:

    ! Splits the MPI_COMM_WORLD for coupling with other codes. This defines the Alya world
    !
    call par_code_split_universe()
    !
    ! Splits the MPI_COMM_WORLD for coupling
    !
    call par_code_split_world()

And read the problem data.

    ! Read problem data
    !
    call reapro()

After doing other initializations, read the domain data.

    !----------------------------------------------------------------------
    !
    ! Read domain data
    !
    !----------------------------------------------------------------------

    call cputim(time1)
    call readom()
    call cputim(time2)
    cpu_start(CPU_READ_DOMAIN) = time2 - time1


### `readom.f90`

This is the file doing calls to read the domain data (such as metadata about the mesh files, the meshfiles themselves).
Here are its location.

    Sources
    |
    |------ kernel
            |
            |------ domain
                    |
                    |------*.f90


It defines the `readom` routine.

    use def_master
    use def_domain
    use def_kermod,       only : kfl_timeline
    use mod_ker_timeline, only : ker_timeline
    implicit none

    if( kfl_timeline /= 0 ) call ker_timeline(0_ip,'INI_READ_MESH')
    !
    ! Open files
    !
    call opfdom(1_ip)
    !
    ! Read geometry data
    !
    if( kfl_examp /= 0 ) then
       call exampl()
    else
       call readim()                         ! Read dimensions: NDIME, NPOIN, NELEM...
       call reastr()                         ! Read strategy:   LQUAD, NGAUS, SCALE, TRANS...
       call cderda()                         ! Derivated param: NTENS, HNATU, MNODE, MGAUS...
       call reageo()                         ! Read arrays:     LTYPE, LNODS, LNODB, LTYPB, COORD...
       call reaset()                         ! Read sets:       LESET, LBSET, LNSET
       call reabcs()                         ! Read bcs:        KFL_FIXNO, KFL_FIXBO, KFL_GEONO
    end if

The two main routines that interest us for the moment are `readim`, which reads the problem dimensions, a kind of metadata file, and `reageo`, which is the routine responsible for parsing the meshfile.

### `reageo.f90`

This is our main interest point. This file parses the mesh file. 
Here are the dependencies (should be taken seriously because of the multiple **side effects** that occur in this part):

    use def_kintyp
    use def_parame
    use def_master 
    use def_domain
    use def_inpout
    use mod_iofile
    use mod_memory
    use def_elmtyp
    use def_kermod, only : multiply_with_curvature

Those are the variables:

    implicit none
    integer(ip)           :: ielem,ipoin,jpoin,inode,idime,iimbo
    integer(ip)           :: iboun,inodb,ielty,ktype,dummi,kfl_defau
    integer(ip)           :: iskew,jskew,jdime,imate,kelem,nlimi
    integer(ip)           :: iblty,knode,kfl_gidsk,kfl_dontr,kpoin
    integer(ip)           :: kfl_binme,ipara,iperi,imast,kfl_icgns
    integer(ip)           :: pelty,ifiel,izone,kfl_ifmat,kfl_elino
    integer(ip)           :: isubd,jstep,jperi
    real(rp)              :: dummr
    character(20)         :: chnod
    integer(ip),  pointer :: lelno(:)

    nullify(lelno)

They are not commented and the meaning of some of them is a bit obscure. 

This instruction ensures a sequential reading, only done by the master:

    if( ISEQUEN .or. ( IMASTER .and. kfl_ptask /= 2 ) ) then

Some other variables, commented this time:

    !
    ! Initializations
    !
    kfl_chege  = 0                   ! Don't check geometry
    kfl_naxis  = 0                   ! Cartesian coordinate system
    kfl_spher  = 0                   ! Cartesian coordinate system
    kfl_bouel  = 0                   ! Element # connected to boundary unknown
    nmate      = 1                   ! No material
    nmatf      = 1                   ! Fluid material
    ngrou_dom  = 0                   ! Groups (for deflated): -2 (slaves), -1 (automatic, master), >0 (master)
    kfl_gidsk  = 0                   ! General skew system
    kfl_dontr  = 0                   ! Read boundaries
    kfl_binme  = 0                   ! =1: Mesh must be read in binary format
    nfiel      = 0                   ! Number of fields
    curvatureDataField = 0           ! curvature data field id
    curvatureField = 0               ! curvature data field id
    kfl_elcoh  = 0                   ! Assume cohesive elements
    kfl_elint  = 0                   ! Assume interface elements
    !
    ! Local variables
    !
    kfl_icgns  = 0                   ! New Alya format for types of elements
    kfl_ifmat  = 0                   ! Materials were not assigned
    ktype      = 0                   ! Element # of nodes is not calculated
    imast      = 0                   ! Master has not been read
    izone      = 0                   ! Zones have not been allocated
    kfl_elino  = 0                   ! Eliminate lost nodes

Now comes the memory allocation, done by `memgeo`. Depending of the value passed in parameter, memgeo allocates a certain amount of memory for a particular topic.
This time again, careful about the side effects and the reading of variables that are not passed in parameter. 
This routine will have to be reworked to ensure the parallelism.

    !
    ! Memory allocation
    !
    call memgeo( 1_ip)               ! Mesh arrays
    call memgeo(49_ip)               ! Zones 
    call memgeo(53_ip)               ! Periodicity

Now starts the reading. First, with some options that are not present in the example file which has been given to me.

    !
    ! Read options and arrays
    ! 
    call ecoute('REAGEO')
    do while( words(1) /= 'GEOME' )
      call ecoute('REAGEO')
    end do
    if( exists('AXISY') ) kfl_naxis = 1
    if( exists('SPHER') ) kfl_spher = 1
    if( exists('CHECK') ) kfl_chege = 1
    if( exists('GID  ') ) kfl_gidsk = 1

    if( exists('ELIMI') ) kfl_elino = 1

The `ecoute` routine is responsible for reading a line of the input file.
_Something I don't get is the purpose of 'REAGEO' string taken as argument. It is not referenced/tested in `ecoute`.
The file that is actually readen by ecoute is actually `nunit`._

Interesting point: this part determines if the file is binary, which means that a binary version of the mesh file already exists.
I should look for its specifications: it could be used as a starting point for defining a parallel compliant format.

    !
    ! Binary file: read/write
    !
    if(exists('BINAR').or.exists('READB')) then
      call geobin(2_ip)
      do while(words(1)/='ENDGE')
         call ecoute('reageo')
      end do
      return
    end if
    if(exists('OUTPU').or.exists('WRITE')) kfl_binar=1

Main loop reading the whole file:

    !
    ! ADOC[0]> $-----------------------------------------------------------------------
    ! ADOC[0]> $ Mesh definition
    ! ADOC[0]> $-----------------------------------------------------------------------
    ! ADOC[0]> GEOMETRY
    !
    do while(words(1)/='ENDGE')
      call ecoute('reageo')

Test to determine if the file is a binary, in this case calls the routine `geobin`.

    if( words(1) == 'BINAR' ) then
      !
      ! Read geometry from binary file
      !
      call geobin(2_ip)

Part parsing the `NODES` section. We'll describe more in details than the previous code section.

Determining if the line parsed contains `NODES`. Note the `words` variable, defined in `def_inpout.f90`, which contains the line content.
_The nodes section is not present in the example file, and the following code description seems to match with the type section as well.
Need to ask for more information..._


    else if( words(1) == 'NODES' ) then
      !
      ! LTYPE: Element types
      !

_I don't understand the purpose of `ktype`._
`nelem` is defined in `def_domain.f90` and is the **number of elements**.

      ktype=nelem

Printing some informations.

    call livinf(27_ip,' ',0_ip)

Iteration over the number of elements

    do ielem=1,nelem

`dummi`'s and `knode` role are commented nowhere.
`nunit` is one of the listen files defined in `def_inpout.f90`
I imagine that, if we take into account the mesh file structure, dummi is the node number and ktype the type.
Since we already know the number of nodes, `dummi` is not readen after having being retrieved (doing the assumption the node are ordered).

       read(nunit,*,err=1) dummi,knode

From the knode identifier, which is a numerical value, we retrieve its real type through `fintyp`, that we store in `ielty`.

       call fintyp(ndime,knode,ielty)

`lexis` informs whether the type _exists_ (is in the file, I guess) or not

       lexis(ielty)=1

Assigns the type to the corresponding element.

       ltype(ielem)=ielty
    end do

Testing the section ends correctly.

    call ecoute('reageo')
    if(words(1)/='ENDNO')&
         call runend('REAGEO: ERROR READING NODES PER ELEMENT, POSSIBLY MISMATCH IN NUMBER OF NODES IN DOM.DAT')
    call mescek(2_ip)

Here comes the types section. This time the parsing in done in a subroutine `reatyp`.

        else if( words(1) == 'TYPES' ) then 
           !
           ! ADOC[1]> TYPES_OF_ELEMENTS [, ALL= char]
           ! ADOC[1]>   int int                                                                     $ Element, type number
           ! ADOC[1]>   ...
           ! ADOC[1]> END_TYPES_OF_ELEMENTS
           ! ADOC[d]> TYPES_OF_ELEMENTS:
           ! ADOC[d]> This field contains the element types. At each line, the first figure is the element number and the second
           ! ADOC[d]> one the element type. If all the elements are the same (say TET04, the following option can be added to the header: 
           ! ADOC[d]> TYPES OF ELEMENTS, ALL=TET04 and then the list should be empty.
           ! ADOC[d]> The correspondance between element type and element number is the following:
           ! ADOC[d]> <ul>
           ! ADOC[d]> <li> 1D elements: </li>
           ! ADOC[d]> <ul>
           ! ADOC[d]> <li> BAR02 =    2 </li> 
           ! ADOC[d]> <li> BAR03 =    3 </li> 
           ! ADOC[d]> <li> BAR04 =    4 </li> 
           ! ADOC[d]> </ul>
           ! ADOC[d]> <li> 2D elements: </li>
           ! ADOC[d]> <ul>
           ! ADOC[d]> <li> TRI03 =   10 </li> 
           ! ADOC[d]> <li> TRI06 =   11 </li> 
           ! ADOC[d]> <li> QUA04 =   12 </li> 
           ! ADOC[d]> <li> QUA08 =   13 </li> 
           ! ADOC[d]> <li> QUA09 =   14 </li> 
           ! ADOC[d]> <li> QUA16 =   15 </li> 
           ! ADOC[d]> </ul>
           ! ADOC[d]> <li> 3D elements: </li>
           ! ADOC[d]> <ul>
           ! ADOC[d]> <li> TET04 =   30 </li> 
           ! ADOC[d]> <li> TET10 =   31 </li> 
           ! ADOC[d]> <li> PYR05 =   32 </li> 
           ! ADOC[d]> <li> PYR14 =   33 </li> 
           ! ADOC[d]> <li> PEN06 =   34 </li> 
           ! ADOC[d]> <li> PEN15 =   35 </li> 
           ! ADOC[d]> <li> PEN18 =   36 </li> 
           ! ADOC[d]> <li> HEX08 =   37 </li> 
           ! ADOC[d]> <li> HEX20 =   38 </li> 
           ! ADOC[d]> <li> HEX27 =   39 </li> 
           ! ADOC[d]> <li> HEX64 =   40 </li> 
           ! ADOC[d]> </ul>
           ! ADOC[d]> <li> 3D 2D-elements: </li>
           ! ADOC[d]> <ul>
           ! ADOC[d]> <li> SHELL =   50 </li> 
           ! ADOC[d]> </ul>
           ! ADOC[d]> </ul>
           !
           call reatyp(kfl_icgns,nelem,ktype,ltype,lexis)

`mescek` checks if the routine is correct. We should'nt need to intervene in this one.
           call mescek(2_ip)

Now, the elements section.

    else if( words(1) == 'ELEME' ) then
       !
       ! ADOC[1]> ELEMENTS 
       ! ADOC[1]>   int int int ...                                                             $ Element, node1, node2, node3 ...
       ! ADOC[1]>   ...
       ! ADOC[1]> END_ELEMENTS 
       ! ADOC[d]> ELEMENTS:
       ! ADOC[d]> This field contains the element connectivity. The first figure is the element number,
       ! ADOC[d]> followed by the list of it nodes.
       !

Printing some informations

       call livinf(28_ip,' ',0_ip)

Ok, now I understand the purpose of `ktype`. I imagine a value equal to 0 means the information is not known and requires a _manual_ parsing.
Since the `ecoute` routine seems very costly (is it not possible to implement several `ecoute`, simpler, and more optimized, by the way?),
the authors want to minimize its usage as possible.

       if( ktype == 0 ) then

Reading a line

          call ecoute('reageo')

Evaluating the end of the section

          do while( words(1) /= 'ENDEL' )

`ielem` is the first column element of the retrieved row, i.e. the element id

             ielem = int(param(1_ip))

Test to see if we don't exceed the maximum element value.

             if( ielem < 0 .or. ielem > nelem ) &
                  call runend('REAGEO: WRONG NUMBER OF ELEMENT '&
                  //adjustl(trim(intost(ielem))))

I guess `knode` is the number of node associated with the element. `nnpar` is another variable modified by `ecoute` through a side-effect.

             knode = nnpar-1

Find the type as a function of knode and the dimension number and associate it with the element in the `ltype` table. See that this process is the same than the one done in the nodes section.

             call fintyp(ndime,knode,ielty)
             ltype(ielem) = ielty

Iterate over the nodes. The table lnods associates each node index, element couple with the corresponding node identifier.

             do inode = 1,nnode(ielty)
                lnods(inode,ielem) = int(param(inode+1))
             end do

Reads another line.

             call ecoute('reageo')
          end do
       else

Basically, does the same process, but reusing the information retrieved in the nodes section parsing.
It does not need the `ecoute` routine and use a simple `read` to fill the `lnods` table, which is much more efficient.

          do ielem = 1,nelem
             ielty = ltype(ielem)
             read(nunit,*,err=2) dummi,(lnods(inode,ielem),inode=1,nnode(ielty))
          end do
          call ecoute('reageo')
       end if
       if( words(1) /= 'ENDEL' )&
            call runend('REAGEO: WRONG ELEMENT FIELD')

Calls the mesh checking.
       call mescek(3_ip)






### `reatyp.f90`

We details here the `reatyp` function which parses the types.

TODO: first part of the code.

This function `ltnew` converts old types to new types. It is very inefficient and must be corrected, since it evaluates all the conditions event if the test is true in the previous one!

    function ltnew(ityol)
      !-----------------------------------------------------------------------
      !
      ! Converts old type to new type
      !
      !-----------------------------------------------------------------------
      use def_kintyp
      integer(ip) :: ltnew,ityol

      if (ityol ==  2) ltnew= 2
      if (ityol ==  3) ltnew= 3
      if (ityol ==  4) ltnew= 4
      if (ityol ==  5) ltnew= 10
      if (ityol ==  6) ltnew= 11
      if (ityol ==  7) ltnew= 12
      if (ityol ==  8) ltnew= 13
      if (ityol ==  9) ltnew= 14
      if (ityol == 10) ltnew= 30
      if (ityol == 11) ltnew= 31
      if (ityol == 12) ltnew= 32
      if (ityol == 13) ltnew= 33
      if (ityol == 14) ltnew= 34
      if (ityol == 15) ltnew= 35
      if (ityol == 16) ltnew= 36
      if (ityol == 17) ltnew= 37
      if (ityol == 18) ltnew= 38
      if (ityol == 19) ltnew= 39

    end function ltnew

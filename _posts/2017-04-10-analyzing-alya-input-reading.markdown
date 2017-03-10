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
I guess `ecoute` reads a line of the input mesh file, but I don't know where it finds the variable specifying the file to read, neither if it's a common variable or something like that._

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
Since we already know the number of nodes, dummi is not readen after having being retrieved (doing the assumption the node are ordered).

       read(nunit,*,err=1) dummi,knode

From the knode identifier, which is a numerical value, we retrieve it's real type through fintyp.

       call fintyp(ndime,knode,ielty)
       lexis(ielty)=1
       ltype(ielem)=ielty
    end do
    call ecoute('reageo')
    if(words(1)/='ENDNO')&
         call runend('REAGEO: ERROR READING NODES PER ELEMENT, POSSIBLY MISMATCH IN NUMBER OF NODES IN DOM.DAT')
    call mescek(2_ip)



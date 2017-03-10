---
layout: post
title: "Parallelizing Alya's Input Reading"
modified: 2017-03-10
categories: 
excerpt:
tags: [Alya]
image:
  feature:
date: 2017-03-10
---

# Analysis of Alya input reading

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

Defines the subroutine `Turnon`. Contains the following dependencies (**important**: it seems that many global variables are used, with multiple side effects! This must be taken into account during the parallelization):

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

This is the file responsible of different calls to read the domain data (such as metadata about the mesh files, the meshfiles themselves).
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


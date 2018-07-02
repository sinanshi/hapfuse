# hapfuse

This is my own fork of the SNPTools::hapfuse tool in [SNPTools v1.0](http://sourceforge.net/projects/snptools).

## Differences to SNPTools::hapfuse

* **Dependency on htslib for parsing input chunks**.
  This allows me to avoid nasty buffer overruns when input VCFs contain too many samples. Also allows for the use of BCF files.

* **Input VCFs must contain the GT, and APP or GP field**.
  APP stands for phred-scaled allelic probability.  SNPTools uses the AP field, which stands for allelic probability.

* **Input VCFs can have fields with arbitrary precision**.  SNPTools::hapfuse uses the AP field with fixed precision floats while hapfuse v0.4 accepts APP and GP fields of arbitrary precision.  

* **Speed and Reliability**.
  Hapfuse is now way faster than SNPTools::hapfuse as it uses htslib for VCF/BCF parsing.  Hapfuse also concurrently loads, fuses and writes chunks, so expect to see CPU usage at about %200.

* **Regression Tests**. Hapfuse comes with regression tests to make sure it works!

## WARNING
Hapfuse does not work with data generated by SNPTools::impute.  This
is because I do not use the AP field and am too lazy to implement
support for it.  Patches welcome.

## Authorship

This code was written and is maintained by Warren Kretzschmar
<warrenk@kth.se>. For bug reports, please raise an issue on the
[bitbucket page](https://bitbucket.org/wkretzsch/hapfuse/issues).

## Prerequisites

### Installation commands for ubuntu 16 LTS

I ran these commands on a fresh ubuntu 16.04 LTS server installation
to get a dynamic and static binary of `hapfuse`:

    sudo apt install perl
    cpan install File::Slurp
    cpan install Test::Files  # only needed for testing
    sudo apt install libboost-all-dev
    sudo apt install zlib1g-dev
    sudo apt install libhts-dev  # installs version 1.2.1 for me

## Installation

This code explains how to build hapfuse v1.1

    # clone the repo
    git clone git@bitbucket.org:wkretzsch/hapfuse.git
    cd hapfuse

    # configure and build hapfuse in hapfuse.1.1
    # the executable will be built at hapfuse.1.1/hapfuse
    ./bootstrap.pl

    # optionally, run regression tests
    cd hapfuse.1.1 && make check

### Compilation issues

#### pthread missing in final link command

I'm not sure why, but it appears that the final link command is
missing `-lpthread`. This should be fixed by adding a pthread
dependency in `configure.ac`. However, if I rerun the final link step
with the addition of `-lpthread` then linking works fine (see for
example static linking commands below). 

#### Static linking

In order to link boost and hts statically I ran
this command instead of the final linking command: 

    /bin/sh ./libtool --tag=CXX   --mode=link g++  -O3 -std=gnu++11 -L/usr/local/lib -Wl,-R,/usr/local/lib /usr/users/flint/winni/lib/libboost_iostreams.a /users/flint/winni/lib/libhts.a /usr/lib64/libz.a /usr/lib64/libbz2.a -o hapfuse hapfuse.o hapSamp.o main.o utils.o -lpthread

When compiling on ubuntu 16.04 LTS I got the final linking step to
work with the command:

    /bin/bash ./libtool  --tag=CXX   --mode=link g++  -O3 -std=gnu++11  -o hapfuse  src/hapfuse.o src/hapSamp.o src/utils.o src/writer.o src/hfHelper.o src/main.o /usr/lib/x86_64-linux-gnu/libhts.a /usr/lib/x86_64-linux-gnu/libz.a  /usr/lib/x86_64-linux-gnu/libboost_iostreams.a /usr/lib/x86_64-linux-gnu/libbz2.a -lpthread

## Pre-compiled binaries

A pre-compiled binary that statically links boost and hts can now be
found in `precompiled-binaries/ubuntu_x86-64/static_boost_hts/`.

## Usage

Fusing three bcf files together and save as gzip compressed bcf

    hapfuse file1.bcf file2.bcf file3.bcf -Ob -o fused.bcf.gz

Fusing three WTCCC style hap/sample file pairs together (see
https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html#hapsample)
and saving as gzip compressed BCF

    cat hap.files
    file1.hap.gz
    file2.hap.gz
    file3.hap.gz

    cat sample.files
    file1.sample
    file2.sample
    file3.sample
    
    # output as bcf
    hapfuse -h hap.files -s sample.files -Ob -o fused.bcf.gz
    
    # since version 1.0: output as WTCCC file pair
    hapfuse -h hap.files -s sample.files -Ow -o fused.wtccc.haps.gz,fused.wtccc.sample

## Description

Hapfuse ligates overlapping haplotype chunks with identical samples
by matching haplotypes at overlap sites.  The -o option is
required. Defaults are given in []. If VCF/BCF files are given on
the command line, then hapfuse assumes that the VCF/BCF files
contain haplotype chunks to ligate.  WTCCC style haplotype chunks
can be provided via the -h and -s arguments. The input and output
format tags are restricted to 'GT' when using the -h and -s
arguments.

### Ligation methods

The 'ligation method' is the method used to determine the diplotype
of a sample at overlapping chunk sites.

#### step

The 'step' method discards the outer half of a chunk's overlap
region haplotypes when determining the diplotype of an overlap
region.  This is the most commonly used method for ligating 
haplotypes.

#### average

The 'average' method uses the average of a sample's two overlapping
diplotypes to determine the new diplotypes. This is the method that
SNPTools::hapfuse uses, but it only makes sense when using GP or APP
fields.

#### linear

The 'linear' method is experimental.  It is the same as average, but
it gives more weight to diplotypes that are closer to a chunk's
center. 

## Thanks

I thank the following people for contributing code to this project:

- Joshua Randall for help with the autoconf files

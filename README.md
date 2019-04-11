# EasyBuild on ShARC config

## Installing Easybuild

Download and install pip (which can then be used to install EasyBuild):

```sh
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py --user
```

Install EasyBuild using pip:

```sh
~/.local/bin/pip install --prefix=/usr/local/community/rse/EasyBuild easybuild
```

Fix problem with `vcs` python package (dependency of EasyBuild):

```sh
touch /usr/local/community/rse/EasyBuild/lib/python2.7/site-packages/vsc/__init__.py
```

Create an EasyBuild modulefile containing TUOS-specific EasyBuild config: 

```sh
mkdir -p /usr/local/community/rse/EasyBuild/modules/all/all 
ln -s /usr/local/community/rse/EasyBuild/tuos/easybuild-modulefile /usr/local/community/rse/EasyBuild/modules/all/all/easybuild
```

Create directories for storing cluster-specific easyconfigs (build specs):

```sh
mkdir -p "/usr/local/community/rse/EasyBuild/tuos/{common,${SGE_CLUSTER_NAME}}/easyconfigs"
```

Switch from using centrally-installed modulefiles to EasyBuild modulefiles:

```sh
module unuse /usr/local/modulefiles
module use /usr/local/community/rse/EasyBuild/modules/all/all
```

Load EasyBuild via a modulefile:

```sh
module load easybuild
```

## Create a customised easyconfig: OpenMPI with Intel OmniPath support

```
cp $EBHOME/easybuild/easyconfigs/o/OpenMPI/OpenMPI-3.1.1-GCC-7.3.0-2.30.eb $EBHOME/tuos/easyconfigs/${SGE_CLUSTER_NAME-sharc}/o/OpenMPI
```

Then modify the new `OpenMPI-3.1.1-GCC-7.3.0-2.30.eb` so that it includes the `osdependencies` and build flags required for Intel OmniPath support.

## Install the 'foss-2018b' toolchain (inc. dependencies)

```sh
eb --parallel=${NSLOTS-1} foss-2018b.eb --robot 
```

This provides specific versions of:

 - GCC
 - OpenMPI
 - OpenBLAS
 - FFTW

## Install other easyconfigs (packages) using the 'foss-2018b' toolchain (inc. dependencies)

```bash
easyconfigs=(
    HDF5-1.10.2-foss-2018b.eb 
    netCDF-4.6.1-foss-2018b.eb 
    netCDF-Fortran-4.4.4-foss-2018b.eb
    Python-2.7.15-foss-2018b.eb
    Python-3.6.6-foss-2018b.eb
    Python-3.7.0-foss-2018b.eb
    Perl-5.28.0-GCCcore-7.3.0.eb
    SAMtools-1.9-foss-2018b.eb
    GROMACS-2018.3-foss-2018b.eb
    HPL-2.2-foss-2018b.eb
)
for easyconfig in ${easyconfigs[@]}; do
    eb --include-easyblocks=$EBHOME/tuos/perl.py --parallel=${NSLOTS-1} $easyconfig --robot 
done
```

NB here we need to use a custom `perl.py` easyblock (build recipe) whilst waiting for a release of the `easybuild-easyblocks` Python package that contains the commits in [this PR](https://github.com/easybuilders/easybuild-easyblocks/pull/1660).

## Install intel toolchain (inc dependencies)

First, copy and customise icc-2018.3.222-GCC-7.3.0-2.30.eb and ifort-2018.3.222-GCC-7.3.0-2.30.eb so the contain the path of the Intel license file.

Next:


```sh
eb --parallel=${NSLOTS-1} intel-2018b.eb --robot 
```

This automatically downloads and installs:

 - Intel compilers
 - Intel MKL
 - Intel MPI

## Install fosscuda-2018b toolchain inc dependencies

First, we copy and customise `CUDA-9.2.88-GCC-7.3.0-2.30.eb` so that it adds `/sbin` to the `path` (needed on Centos 7.x so that `ldconfig` can be found by all users).

Next, copy and customise `OpenMPI-3.1.1-gcccuda-2018b.eb` so that it includes PSM2 (OPA) support. 

**TODO**: finish and test

easyconfigs=(
    fosscuda-2018b.eb
    #cuDNN-7.1.4.18-fosscuda-2018b.eb
)
for easyconfig in ${easyconfigs[@]}; do
    eb --include-easyblocks=$EBHOME/tuos/perl.py --parallel=${NSLOTS-1} $easyconfig --robot 
done

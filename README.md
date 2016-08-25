# LKMC README file

## Requirements

This code requires [Python 2.7](http://www.python.org/download/releases/2.7/) to run.  It also
currently requires the Intel compilers and gcc to compile the C and Fortran libraries.

The software [nauty](http://pallini.di.uniroma1.it/) is included in the package and requires the
GNU autoconf installation system.

### Python modules

This is a list of third-party python libraries that this code requires to run:

* [NumPy](http://www.numpy.org/)
* [SciPy](http://www.scipy.org/) (>= 0.12.0)
* [Matplotlib](http://matplotlib.org/)
* [Sphinx](http://sphinx-doc.org/) (for building the documentation)
* [numpydoc](https://pypi.python.org/pypi/numpydoc) (for building the documentation)
* [nose](https://nose.readthedocs.org/en/latest/) (for running the tests)

These libraries can be installed in a number of ways.  For example, you may wish to use a package
manager such as [MacPorts](http://www.macports.org/) on Mac OS X or
[apt-get](http://en.wikipedia.org/wiki/Advanced_Packaging_Tool) on Debian. Alternatively you could
use the Python package manager ([pip](https://pypi.python.org/pypi/pip)) or install the libraries
from source.

### Forces library

For the force calculations we require that an external program be installed that we can call from
our Python code. Currently we support the following codes:

* LBOMD (via the Python interface)
* [LAMMPS](http://lammps.sandia.gov/doc/Section_python.html) (via the Python/shared library
  interface)
* [SIESTA](http://departments.icmab.es/leem/siesta/) (built as an executable and installed in your
  PATH)

__Note__: You need to install the force library on all machines you intend to run on (i.e. all
those that appear in the nodes file).

### Additional programmes

The following additional programmes are also required:

* [POV-Ray](http://www.povray.org/) (optional; to create images of the defects in the system at
  each step or per transition)

## Mac OS X and MacPorts Python

You may need to add a symbolic link to /Library/Frameworks to make sure that the MacPortsPython takes
precedence over the system Python when using f2py.  This command will achieve this (you will
probably need to be root):

```sh
ln -s /opt/local/Library/Frameworks/Python.framework /Library/Frameworks/Python.framework
```

## Installation

There are two options for installing LKMC:

* build the code in-place and add the main directory to you PATH and PYTHONPATH.
* install it into your Python site-packages directory, for example using a
  [virtualenv](http://docs.python-guide.org/en/latest/dev/virtualenvs/).

__Note__: You need to install the code using one of the above methods on all machines you intend to
run on (i.e. all those that appear in the nodes file).

### Build in-place

You should add the top LKMC directory (eg. `$HOME/git/LKMC`) to your PATH and PYTHONPATH, eg. for
bash:

```sh
export PYTHONPATH=$HOME/git/LKMC:$PYTHONPATH
```
or for tcsh:

```sh
setenv PYTHONPATH $HOME/git/LKMC:$PYTHONPATH
```
and the same for PATH. These lines should be added to your shell startup files, for example
".profile", ".bashrc" or ".tcshrc".

You should then copy and edit the setup.cfg file:

```sh
cp setup.cfg.example setup.cfg
```

There are comments in the file to help with choosing settings. For example, to use the Intel C
compiler you could uncomment the line in the 'build_ext' section that says `compiler = intelem`.

You should then run setup:

```sh
python setup.py build_ext --inplace
```

which should compile all the extensions.

### Install Python package

First you should copy and edit the setup.cfg file:

```sh
cp setup.cfg.example setup.cfg
```

There are comments in the file to help with choosing settings. For example, to use the Intel C
compiler you could uncomment the line in the 'build_ext' section that says `compiler = intelem`.

You should then build and install the package (before installing you should run the test suite, see
below):

```sh
python setup.py build
python setup.py install
```

You should then be able to run the code, as detailed in the 'Execution' section below.


### Tests

You should run the test suite to ensure everything is working. You should also clone
the 'LKMCData' repository and place it in the same directory as this repository.
If you don't do this many of the (important) tests will be skipped.

Fast/unit tests are run using:

```sh
nosetests LKMC/tests -v
```

Slower, more complex, tests are run using:

```sh
nosetests LKMC/slow_tests -v
```

Alternatively, you can run both the fast/unit tests and slow tests by running:

```sh
nosetests LKMC -v
```

All the tests should exit successfully.

## Building the Documentation

To build the html documentation you should run (after copying setup.cfg.example as described
above):

```sh
python setup.py build_sphinx
```

Following this the documentation can be found in `doc/output/html/` (open
`doc/output/html/index.html` in your browser).

__Note__: at the time of writing the html documentation was out of date/partially complete.

## Nodes file

If you're planning to run a KMC simulation (instead of just do a minimisation, for example) you
need to create a nodes file, to tell the code where to run.  Each line of the file should contain a
hostname for each process to run on (similar to an MPI machinefile).  Blank lines and lines
beginning with '#' are ignored. For example, a file containing only the lines:

```
localhost
localhost
localhost
localhost
```

will run on 4 processors on your machine.

In order to automatically generate such a file on hydra or hera you could add the following lines
to your init scripts:

```
cp -f $PBS_NODEFILE nodes.IN
```

for hydra (PBS), or:

```
srun /bin/hostname | sort > nodes.IN
```

for hera (slurm).

__Note__: with the above nodes file the code will actually run on 4+2 processors.  The lines in the
nodes file are for worker processes.  In addition to those we have the main (server) process and
also a writer process, which handles IO intensive tasks (writing files, zipping files, plotting
stuff, rendering images) that are carried out asynchronously. That being said, you should set the
number of lines in the nodes file to the number of available processors; the additional 2 processes
are not CPU intensive.

__Important__: you must ensure that you can SSH into any node that you specify in the nodes file
without requiring a password (public key login must be configured for those nodes).

## Input files

If you're planning to run a KMC simulation (instead of just doing a minimisation, for example) you
need to ensure the input files exist and are named correctly.  When running other types of
simulations (minimisations, etc.) the input files are specified as command line arguments.  

The input directory specified in the input file (parameter name 'inputDirectory'; defaults to
'input') must exist.  A reference lattice file named 'ref.dat' and the initial lattice file named
'launch.dat' must exist in the input directory.  

In addition you have a directory containing any input files that are required by the MD library you
are using. The name of this directory is set in the input file and defaults to 'md'.

Your directory structure should look something like this for running a KMC simulation:

```
runDir/
runDir/lkmcInput.IN
runDir/nodes.IN
runDir/input/
runDir/input/ref.dat
runDir/input/launch.dat
runDir/md/
runDir/md/*MDInputFiles*
```

## Execution

The only script that should ever be called directly is "lkmc.py". No other file
should be called.  Examples:

 - Print the version:

```sh
lkmc.py -v
``` 

 - To run a minimisation you would run:

```sh
lkmc.py Minimise [OPTIONS] [ARGS]
```
   where a list of options/args can be printed by running:

```sh
lkmc.py Minimise -h
```

 - To run a KMC simulation:

```sh
lkmc.py KMC [OPTIONS] [ARGS]
```
 where a list of options/args can be printed by running:

```sh
lkmc.py KMC -h
```

 - All available options to the lkmc.py wrapper script are available by running:

```sh
lkmc.py -h
```

## Files

Here we list all the files that make up the LKMC project.

### Python source code

./LKMC/__init__.py\s\s
./LKMC/Algebra.py\s\s
./LKMC/ART.py\s\s
./LKMC/Atoms.py\s\s
./LKMC/Basin.py\s\s
./LKMC/clibs/__init__.py
./LKMC/clibs/defects.py
./LKMC/clibs/graphs.py
./LKMC/clibs/io_module.py
./LKMC/clibs/lanczos.py
./LKMC/clibs/lattice.py
./LKMC/clibs/nauty_iface.py
./LKMC/clibs/neb.py
./LKMC/clibs/numpy_utils.py
./LKMC/clibs/setup.py
./LKMC/clibs/utilities.py
./LKMC/clibs/vectors.py
./LKMC/Client.py
./LKMC/ClientServerInterface.py
./LKMC/Constants.py
./LKMC/Defects.py
./LKMC/Deposition.py
./LKMC/Dimer.py
./LKMC/Dummy.py
./LKMC/ExternalEvents.py
./LKMC/Filtering.py
./LKMC/Forces.py
./LKMC/Globals.py
./LKMC/Graphs.py
./LKMC/hooks/__init__.py
./LKMC/hooks/addRemoveVacs.py
./LKMC/hooks/example_hook.py
./LKMC/hooks/hook_utils.py
./LKMC/Images.py
./LKMC/Input.py
./LKMC/KMC.py
./LKMC/Lanczos.py
./LKMC/lapack/__init__.py
./LKMC/lapack/setup.py
./LKMC/Lattice.py
./LKMC/Lbomd.py
./LKMC/Minimise.py
./LKMC/MinModeFollower.py
./LKMC/NEB.py
./LKMC/Plot.py
./LKMC/Prefactor.py
./LKMC/Profiling.py
./LKMC/RAT.py
./LKMC/Rates.py
./LKMC/Refine.py
./LKMC/Rendering.py
./LKMC/Results.py
./LKMC/Search.py
./LKMC/Server.py
./LKMC/setup.py
./LKMC/Status.py
./LKMC/String.py
./LKMC/Timer.py
./LKMC/updateParameters.py
./LKMC/Utilities.py
./LKMC/Vectors.py
./LKMC/Version.py
./LKMC/Writer.py

### Python unit tests

./LKMC/slow_tests/__init__.py
./LKMC/slow_tests/test_Minimise.py
./LKMC/tests/__init__.py
./LKMC/tests/base.py
./LKMC/tests/test_Defects.py
./LKMC/tests/test_Forces.py
./LKMC/tests/test_Graphs.py
./LKMC/tests/test_Lattice.py
./LKMC/tests/test_multiprocessing.py
./LKMC/tests/test_Utilities.py
./LKMC/tests/test_Vectors.py

### Python setup script and example configuration

./setup.py
./setup.cfg.example

### Main Python script for running the program

./lkmc.py

### Python utility scripts

./scripts/archive.py
./scripts/convertOldHashKeyVolumes.py
./scripts/convertVolumeFile.py
./scripts/debugPrefactorCalc.py
./scripts/fixTransitionInput.py
./scripts/isolateVolume.py
./scripts/mergeLKMCInputFile.py
./scripts/retrieveLatticesFromCDT.py

### Example simulations





### C source code

./LKMC/clibs/boxeslib.c
./LKMC/clibs/boxeslib.h
./LKMC/clibs/defects.c
./LKMC/clibs/defects.h
./LKMC/clibs/graphs.c
./LKMC/clibs/graphs.h
./LKMC/clibs/io_module.c
./LKMC/clibs/io_module.h
./LKMC/clibs/lanczos.c
./LKMC/clibs/lanczos.h
./LKMC/clibs/lattice.c
./LKMC/clibs/lattice.h
./LKMC/clibs/nauty_iface.c
./LKMC/clibs/nauty_iface.h
./LKMC/clibs/neb.c
./LKMC/clibs/neb.h
./LKMC/clibs/utilities.c
./LKMC/clibs/utilities.h
./LKMC/clibs/vectors.c
./LKMC/clibs/vectors.h

### Nauty source code

./LKMC/clibs/nauty/addedgeg.c
./LKMC/clibs/nauty/amtog.c
./LKMC/clibs/nauty/biplabg.c
./LKMC/clibs/nauty/bliss2dre.c
./LKMC/clibs/nauty/blisstog.c
./LKMC/clibs/nauty/callgeng.c
./LKMC/clibs/nauty/catg.c
./LKMC/clibs/nauty/changes24-25.txt
./LKMC/clibs/nauty/checks6.c
./LKMC/clibs/nauty/complg.c
./LKMC/clibs/nauty/config.guess
./LKMC/clibs/nauty/config.sub
./LKMC/clibs/nauty/config.txt
./LKMC/clibs/nauty/configure
./LKMC/clibs/nauty/configure.ac
./LKMC/clibs/nauty/copyg.c
./LKMC/clibs/nauty/deledgeg.c
./LKMC/clibs/nauty/directg.c
./LKMC/clibs/nauty/dreadnaut.c
./LKMC/clibs/nauty/dretog.c
./LKMC/clibs/nauty/formats.txt
./LKMC/clibs/nauty/genbg.c
./LKMC/clibs/nauty/geng.c
./LKMC/clibs/nauty/genrang.c
./LKMC/clibs/nauty/gentourng.c
./LKMC/clibs/nauty/gtnauty.c
./LKMC/clibs/nauty/gtools-h.in
./LKMC/clibs/nauty/gtools.c
./LKMC/clibs/nauty/gutil1.c
./LKMC/clibs/nauty/gutil2.c
./LKMC/clibs/nauty/gutils.h
./LKMC/clibs/nauty/install-sh
./LKMC/clibs/nauty/labelg.c
./LKMC/clibs/nauty/linegraphg.c
./LKMC/clibs/nauty/listg.c
./LKMC/clibs/nauty/makefile.basic
./LKMC/clibs/nauty/makefile.in
./LKMC/clibs/nauty/multig.c
./LKMC/clibs/nauty/naucompare.c
./LKMC/clibs/nauty/naugraph.c
./LKMC/clibs/nauty/naugroup.c
./LKMC/clibs/nauty/naugroup.h
./LKMC/clibs/nauty/naurng.c
./LKMC/clibs/nauty/naurng.h
./LKMC/clibs/nauty/nausparse.c
./LKMC/clibs/nauty/nausparse.h
./LKMC/clibs/nauty/nautaux.c
./LKMC/clibs/nauty/nautaux.h
./LKMC/clibs/nauty/nautest.c
./LKMC/clibs/nauty/nautest1.dre
./LKMC/clibs/nauty/nautest1a.ans
./LKMC/clibs/nauty/nautest1a.dre
./LKMC/clibs/nauty/nautest1b.ans
./LKMC/clibs/nauty/nautest1b.dre
./LKMC/clibs/nauty/nautest1c.ans
./LKMC/clibs/nauty/nautest1c.dre
./LKMC/clibs/nauty/nautest2.c
./LKMC/clibs/nauty/nautest2.dre
./LKMC/clibs/nauty/nautest2a.ans
./LKMC/clibs/nauty/nautest2b.ans
./LKMC/clibs/nauty/nautest2c.ans
./LKMC/clibs/nauty/nautesta.ans
./LKMC/clibs/nauty/nautestb.ans
./LKMC/clibs/nauty/nautestc.ans
./LKMC/clibs/nauty/nauthread1.c
./LKMC/clibs/nauty/nauthread2.c
./LKMC/clibs/nauty/nautil.c
./LKMC/clibs/nauty/nautinv.c
./LKMC/clibs/nauty/nautinv.h
./LKMC/clibs/nauty/naututil-h.in
./LKMC/clibs/nauty/naututil.c
./LKMC/clibs/nauty/nauty-h.in
./LKMC/clibs/nauty/nauty.c
./LKMC/clibs/nauty/nautyex1.c
./LKMC/clibs/nauty/nautyex10.c
./LKMC/clibs/nauty/nautyex2.c
./LKMC/clibs/nauty/nautyex3.c
./LKMC/clibs/nauty/nautyex4.c
./LKMC/clibs/nauty/nautyex5.c
./LKMC/clibs/nauty/nautyex6.c
./LKMC/clibs/nauty/nautyex7.c
./LKMC/clibs/nauty/nautyex8.c
./LKMC/clibs/nauty/nautyex9.c
./LKMC/clibs/nauty/newedgeg.c
./LKMC/clibs/nauty/NRswitchg.c
./LKMC/clibs/nauty/nug25.pdf
./LKMC/clibs/nauty/planarg.c
./LKMC/clibs/nauty/planarity.c
./LKMC/clibs/nauty/planarity.h
./LKMC/clibs/nauty/ranlabg.c
./LKMC/clibs/nauty/README
./LKMC/clibs/nauty/README_24
./LKMC/clibs/nauty/rng.c
./LKMC/clibs/nauty/rng.h
./LKMC/clibs/nauty/runalltests
./LKMC/clibs/nauty/schreier.c
./LKMC/clibs/nauty/schreier.h
./LKMC/clibs/nauty/schreier.txt
./LKMC/clibs/nauty/shortg.c
./LKMC/clibs/nauty/showg.c
./LKMC/clibs/nauty/sorttemplates.c
./LKMC/clibs/nauty/splay.c
./LKMC/clibs/nauty/subdivideg.c
./LKMC/clibs/nauty/sumlines.c
./LKMC/clibs/nauty/testg.c
./LKMC/clibs/nauty/traces.c
./LKMC/clibs/nauty/traces.h
./LKMC/clibs/nauty/watercluster2.c

### LAPACK wrappers

./LKMC/lapack/dstev.f
./LKMC/lapack/dstevx.f
./LKMC/lapack/dsyev.f
./LKMC/lapack/dsyevx.f
./LKMC/lapack/ilaenv.f

### This file and the license

./README.md
./LICENSE.md

### Files for building the documentation

./doc/conf.py
./doc/contributing.rst
./doc/examples/examples.rst
./doc/gen_lkmc_modules.sh
./doc/index.rst
./doc/installation.rst
./doc/license/license.rst
./doc/LKMC/LKMC.Algebra.rst
./doc/LKMC/LKMC.ART.rst
./doc/LKMC/LKMC.Atoms.rst
./doc/LKMC/LKMC.Basin.rst
./doc/LKMC/LKMC.clibs.defects.rst
./doc/LKMC/LKMC.clibs.graphs.rst
./doc/LKMC/LKMC.clibs.io_module.rst
./doc/LKMC/LKMC.clibs.lanczos.rst
./doc/LKMC/LKMC.clibs.lattice.rst
./doc/LKMC/LKMC.clibs.neb.rst
./doc/LKMC/LKMC.clibs.numpy_utils.rst
./doc/LKMC/LKMC.clibs.rst
./doc/LKMC/LKMC.clibs.setup.rst
./doc/LKMC/LKMC.clibs.utilities.rst
./doc/LKMC/LKMC.clibs.vectors.rst
./doc/LKMC/LKMC.Client.rst
./doc/LKMC/LKMC.ClientServerInterface.rst
./doc/LKMC/LKMC.Constants.rst
./doc/LKMC/LKMC.Defects.rst
./doc/LKMC/LKMC.Deposition.rst
./doc/LKMC/LKMC.Dimer.rst
./doc/LKMC/LKMC.Dummy.rst
./doc/LKMC/LKMC.ExternalEvents.rst
./doc/LKMC/LKMC.Filtering.rst
./doc/LKMC/LKMC.Forces.rst
./doc/LKMC/LKMC.Globals.rst
./doc/LKMC/LKMC.Graphs.rst
./doc/LKMC/LKMC.hooks.addRemoveVacs.rst
./doc/LKMC/LKMC.hooks.example_hook.rst
./doc/LKMC/LKMC.hooks.hook_utils.rst
./doc/LKMC/LKMC.hooks.rst
./doc/LKMC/LKMC.Images.rst
./doc/LKMC/LKMC.Input.rst
./doc/LKMC/LKMC.KMC.rst
./doc/LKMC/LKMC.Lanczos.rst
./doc/LKMC/LKMC.lapack.rst
./doc/LKMC/LKMC.lapack.setup.rst
./doc/LKMC/LKMC.Lattice.rst
./doc/LKMC/LKMC.Lbomd.rst
./doc/LKMC/LKMC.Minimise.rst
./doc/LKMC/LKMC.MinModeFollower.rst
./doc/LKMC/LKMC.NEB.rst
./doc/LKMC/LKMC.Plot.rst
./doc/LKMC/LKMC.Prefactor.rst
./doc/LKMC/LKMC.Profiling.rst
./doc/LKMC/LKMC.RAT.rst
./doc/LKMC/LKMC.Rates.rst
./doc/LKMC/LKMC.Refine.rst
./doc/LKMC/LKMC.Rendering.rst
./doc/LKMC/LKMC.Results.rst
./doc/LKMC/LKMC.rst
./doc/LKMC/LKMC.Search.rst
./doc/LKMC/LKMC.Server.rst
./doc/LKMC/LKMC.setup.rst
./doc/LKMC/LKMC.Status.rst
./doc/LKMC/LKMC.String.rst
./doc/LKMC/LKMC.Timer.rst
./doc/LKMC/LKMC.Utilities.rst
./doc/LKMC/LKMC.Vectors.rst
./doc/LKMC/LKMC.Version.rst
./doc/LKMC/LKMC.Writer.rst
./doc/LKMC/modules.rst
./doc/make.bat
./doc/Makefile
./doc/parameters/art.rst
./doc/parameters/barrierSearch.rst
./doc/parameters/basin.rst
./doc/parameters/defects.rst
./doc/parameters/deposition.rst
./doc/parameters/dimer.rst
./doc/parameters/filtering.rst
./doc/parameters/hooks.rst
./doc/parameters/lammps.rst
./doc/parameters/lammps_short_instruction.rst
./doc/parameters/lanczos.rst
./doc/parameters/lbfgs.rst
./doc/parameters/llminmode.rst
./doc/parameters/main.rst
./doc/parameters/minimisation.rst
./doc/parameters/minmode.rst
./doc/parameters/neb.rst
./doc/parameters/output.rst
./doc/parameters/parameters.rst
./doc/parameters/rat.rst
./doc/parameters/rates.rst
./doc/parameters/refine.rst
./doc/parameters/server.rst
./doc/parameters/string.rst
./doc/parameters/volumes.rst
./doc/running.rst

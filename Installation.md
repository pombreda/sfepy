

# Installation #
## Requirements ##

Installation prerequisites:
  * recent numpy, scipy (with umfpack wrapper, or umfpack scikit), swig

Dependencies:
  * matplotlib, pyparsing, umfpack, pytables
  * schroedinger.py requires pysparse, gmsh (2D) , tetgen (3D)
  * log.py (live plotting) requires multiprocessing, matplotlib with GTKAgg
  * isfepy requires ipython, matplotlib with WXAgg
  * postproc.py requires mayavi2

SfePy is known to work on various flavours of Linux, on Intel Macs and Windows.

On Linux, consult the package manager of your favourite distribution.
For example in Gentoo:
> $ emerge -va pytables pyparsing numpy scipy matplotlib ipython
in Debian:
> $ apt-get install python-tables python-pyparsing python-matplotlib python-scipy

On Windows, all the required packages are part of the Enthought Python
Distribution (EPD) (1) which is free for academic purposes. A completely
free Python(x,y) (2) can be used too, but pyparsing has to be installed
manually.

  * (1) http://www.enthought.com/products/epd.php
  * (2) http://www.pythonxy.com/foreword.php

SfePy can be used without any installation by running the scripts from the top-level directory of the distribution (TOPDIR), or can be installed locally or system-wide.

SfePy should work both with bleeding edge (SVN) and last released versions of its dependencies, see INSTALL file in the tarball. Submit an issue, please, in case this does not hold.

## Generic installation instruction ##

Download the [latest source release](http://code.google.com/p/sfepy/downloads/list) or the development version from our git repository http://git.sympy.org/?p=sfepy.git, see the [Downloads](http://code.google.com/p/sfepy/wiki/Downloads?tm=2) tab.

#### In-place compilation of C extension modules ####
Using the Makefile (Linux, Mac OS X):
  1. Look at site\_cfg\_template.py and follow the instructions therein.
  1. (Optionally) edit the Makefile:
> > - compiler, swig command, ...
  1. 'make'

Using distutils (all platforms):

> python setup.py build\_ext --inplace

#### Installation ####

  * system-wide: 'python setup.py install'
  * local: 'python setup.py install --root=<installation prefix>'

See also INSTALL and RELEASE\_NOTES files in the tarball.

If all went well, proceed with [running tests and examples](BasicTasks.md).

## Check the Sfepy installation ##

After installing sfepy you can chek if all the functionlities are working by running the automated test. From the source dir type:

```
./runTests.py
```

If some test fails, please run it in debug mode:

```
./runTests.py --debug test/failing_test_name.py
```

and report the output to the `sfepy-devel` mailing list.

# Platform-specific notes #


## Using umpfpack on fedora 8 ##

(contributed by David Huard)

> entry in numpy site.cfg:

```
  [umfpack]
  library_dirs=/usr/lib64
  include_dirs = /usr/include/suitesparse
```

> Comment by david.huard,  Mar 26, 2008:

> Of course, suitesparse and suitesparse-devel must be installed.


## Intel Mac ##
(thanks to Dominique Orban for his advice)

> To build SfePy on an Intel Mac the following options need to be set in site\_cfg.py:

> opt\_flags = '-g -O2 -fPIC -DPIC -fno-strict-aliasing -fno-common -dynamic'
> link\_flags = '-dynamiclib -undefined dynamic\_lookup -fPIC -DPIC'

> (revision http://hg.sympy.org/sfepy/rev/609196c918be is needed)

## Installation on Ubuntu (tested on Jaunty Jackalope 9.04) ##
### Prerequisites ###

Firstly you have to install the dependencies packages:
```
sudo aptitude install python-scipy python-matplotlib python-tables python-pyparsing libsuitesparse-dev 
```

Then download and install the `umfpack scikit`s in some local dir. In the following example it will be installed in `$HOME/local`:

```
svn checkout http://svn.scipy.org/svn/scikits/trunk/umfpack
cd umfpack
mkdir -p ${HOME}/local/lib/python2.6/site-packages
python setup.py install --prefix=${HOME}/local
```

Add to your `.bashrc` the line:

```
export PYTHONPATH="${HOME}/local"
```

then re-open a terminal and if the scikits was installed correctly importing `scikits.umfpack` in python should give no error:

```
$ python
>>> import scikits.umfpack
>>> 
```

### Installing Sfepy ###

Now [download](Downloads.md) the latest **sfepy** tarball release (or the latest development version from [this link](http://git.sympy.org/?p=sfepy.git;a=snapshot;h=HEAD;sf=tgz)).

Uncompress the archive and enter the sfepy dir, then type

```
make
```

after a few minutes the compilation finishes.

Finally you can test sfepy with:

```
./runTests.py
```

If some test fails see [Check the Sfepy installation](#Check_the_Sfepy_installation.md) section for further details.
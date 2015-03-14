# Basic Tasks #

Jump to:
  * [quick start](BasicTasks#Quick_start.md)
  * [running tests](BasicTasks#Running_tests.md)
  * [computations and examples](BasicTasks#Computations_and_examples.md)

## Quick start ##

Basic usage:
  * `$ ./simple.py input/poisson.py` ... creates simple.vtk
  * `$ ./simple.py input/pfdpm_permeability.py` ... creates perf\_symm944t.vtk
  * `$ ./runTests.py` ... see [running tests](BasicTasks#Running_tests.md)
  * `$ ./isfepy` ... follow the help printed on its startup

Surface extraction:
  * `$ ./findSurf.py database/t.1.node -` ... creates surf\_t.1.mesh

Applications:
  * phononic materials:
    * `$ ./eigen.py -p input/phono/band_gaps.py` ... see input/phono/output/
  * schroedinger.py (order is important below):
    1. `$ ./schroedinger.py --2d --mesh`
    1. `$ ./schroedinger.py --2d --hydrogen`
    1. `$ ./postproc.py mesh.vtk`

Stand-alone examples:
  * `$ python examples/rs_correctors.py`
  * `$ python examples/compare_elastic_materials.py`
  * `$ python examples/live_plot.py`

## Running tests ##

The tests are run by the 'runTests.py' script.

```
$ ./runTests.py -h

usage: runTests.py [options] [testFileName]

options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  -d directory, --dir=directory
                        directory with tests [default: tests]
  -o directory, --output=directory
                        directory for storing test results and temporary files
                        [default: output-tests]
  --debug               raise silenced exceptions to see what was wrong
                        [default:False]
  --filter              filter output (suppress all except test messages)
                        [default:False]
  --filter-more         filter output (suppress all except test result
                        messages) [default: False]
```

### Common tasks ###

  * run all tests, filter output; result files related to the tests can be found in output-tests directory:
```
./runTests.py
./runTests.py --filter-more
./runTests.py --filter-less
```

  * run a particular test file, filter output
```
./runTests.py tests/test_input_le.py # Test if linear elasticity input file works.
```

  * debug a failing test
```
./runTests.py tests/test_input_le.py --debug
```

## Computations and examples ##

  * The example problems in the 'input' directory can be computed by the script 'simple.py' which is in the top-level directory of the distribution. If it is run without arguments, a help message is printed:
```
$ ./simple.py
usage: simple.py [options] filename_in

options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  -o filename           basename of output file(s) [default: <basename of
                        input file>]
  --format=format       output file format, one of: {vtk, h5, mesh} [default:
                        vtk]
  --save-ebc            save problem state showing EBC (Dirichlet conditions)
  --save-regions        save problem regions as meshes
  --save-field-meshes   save meshes of problem fields (with extra DOF nodes)
  --save-region-field-meshes
                        save meshes of regions of problem fields (with extra
                        DOF nodes)
  --solve-not           do not solve (use in connection with --save-*)
  --list=what           list data, what can be one of: {terms}
```

  * Additional (stand-alone) examples are in the examples/ directory, e.g.:
```
$ python examples/compare_elastic_materials.py
```

  * Parametric study example:
```
$ ./simple.py input/poisson_parametric_study.py
```

### Common tasks ###

  * run a simulation:
```
./simple.py input/poisson.py
./simple.py input/poisson.py -o some_results # -> produces some_results.vtk
```

  * print available terms:
```
./simple.py --list=terms
```

  * run a simulation and also save Dirichlet boundary conditions:
```
./simple.py --save-ebc input/poisson.py # -> produces an additional .vtk file with BC visualization
```
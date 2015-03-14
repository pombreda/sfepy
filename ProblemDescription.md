# Problem Description #

Here we discuss the basic items that users have to specify in their input files.
For complete examples, see the problem description files in the 'input/' directory of SfePy.

Jump to:
  * [FE mesh](ProblemDescription#FE_mesh.md)
  * [regions](ProblemDescription#Regions.md)
  * [fields, variables and integrals](ProblemDescription#Fields_variables_and_integrals.md)
  * [boundary conditions](ProblemDescription#Boundary_conditions.md)
  * [initial conditions](ProblemDescription#Initial_conditions.md)
  * [materials](ProblemDescription#Materials.md)
  * [equations and terms](ProblemDescription#Equations_and_terms.md)
  * [configuring solvers](ProblemDescription#Configuring_solvers.md)
  * [functions](ProblemDescription#Functions.md)
  * [miscellaneous](ProblemDescription#Miscellaneous.md)

## FE mesh ##

A FE mesh defining a domain geometry can be stored in several formats:
  * legacy VTK (`.vtk`)
  * medit mesh file (`.mesh`)
  * tetgen mesh files (`.node, .ele`)
  * comsol text mesh file (`.txt`)
  * custom HDF5 file (`.h5`)
    * example:
```
filename_mesh = 'database/a_mesh.vtk'
```

## Regions ##

Regions serve to select a certain part of the computational domain (= selection of nodes and elements of a FE mesh). They are used to define the boundary conditions, the domains of terms and materials etc.

  * Region selection syntax:

> Entity selections:
    * all
    * nodes of surface
    * nodes in 

&lt;expr&gt;


    * nodes by 

&lt;function&gt;


    * node 

&lt;id&gt;


    * elements of group 

&lt;integer&gt;


    * elements by 

&lt;efunction&gt;


    * r.<name of another region>

> Notation:
    * 

&lt;expr&gt;

 is a logical expression like `(y <= 0.00001) & (x < 0.11)`
    * 

&lt;function&gt;

 is e.g. `afunction( x, y, z, otherArgs )`
    * 

&lt;efunction&gt;

 is e.g. `efunction( domain )`

> Region operations:
    * node-wise: `+n, -n, *n` (union, set difference, intersection)
    * element-wise: `+e, -e, *e` (union, set difference, intersection)

  * Region definition syntax:

> Long syntax: a region is defined by the following dictionary ([.md](.md) denote optional keys):

```
region_<number> = {
    'name' : <string>,
    'select' : <selection string>,
    ['forbid'] : group <integer>[, <integer>]* # forbid elements of listed groups
    ['can_cells'] : <boolean> # determines whether a region can have cells (volume in 3D)
}
```

> Example definitions:

```
    region_20 = {
        'name' : 'Left',
        'select' : 'nodes in (x < -0.499)'
    }
    region_21 = {
        'name' : 'Right',
        'select' : 'nodes in (x > 0.499)'
    }
    region_31 = {
        'name' : 'Gamma1',
        'select' : """(elements of group 1 *n elements of group 4)
                      +n
                      (elements of group 2 *n elements of group 4)
                      +n
                      ((r.Left +n r.Right) *n elements of group 4)
                   """,
        'forbid' : 'group 1 2'
    }
```

> Example definitions, short syntax:

```
    regions = {
        'Left' : ('nodes in (x < -0.499)', {}),
        'Right' : ('nodes in (x > 0.499)', {}),
        'Gamma1' : ("""(elements of group 1 *n elements of group 4)
                       +n
                       (elements of group 2 *n elements of group 4)
                       +n
                       ((r.Left +n r.Right) *n elements of group 4)""",
                     {'forbid' : 'group 1 2'}),
    }
```

## Fields variables and integrals ##
  * fields correspond to FE spaces
    * example: P1 elements in 2D on a whole domain Omega
```
 field_1 = {
    'name' : 'temperature',
    'dim' : (1,1),
    'domain' : 'Omega',
    'bases' : {'Omega' : '2_3_P1'}
}
```
  * variables use the FE approximation given by the specified field:
    * example, long syntax:
```
variable_1 = {
    'name' : 't',
    'kind' : 'unknown field',
    'field' : 'temperature',
    'order' : 0, # order in the global vector of unknowns
}

variable_2 = {
    'name' : 's',
    'kind' : 'test field',
    'field' : 'temperature',
    'dual' : 't',
}
```
    * example, short syntax:
```
variables = {
    't' : ('unknown field', 'temperature', 0),
    's' : ('test field', 'temperature', 't'),
}
```
  * integrals (quadrature rules):
```
integral_1 = {
    'name' : 'i1',
    'kind' : 'v',
    'quadrature' : 'gauss_o2_d2', # <quadrature name>
}

import numpy as nm
N = 2
integral_2 = {
    'name' : 'i2',
    'kind' : 'v',
    'quadrature' : 'custom', # <quadrature name>
    'vals'    : zip(nm.linspace( 1e-10, 0.5, N ),
                    nm.linspace( 1e-10, 0.5, N )),
    'weights' : [1./N] * N,
}
```
    * available quadratures are in sfe/fem/quadratures.py - it is still preliminary and incomplete
    * naming convention: 

&lt;family&gt;

_o_

&lt;order&gt;

_d_

&lt;dimension&gt;



## Boundary conditions ##

  * Dirichlet (essential) boundary conditions, long syntax:
```
ebc_<number> = {
    'name' : <string>,
    'region' : <region_name>,
    'dofs' : {<dof_specification> : <value>[,
              <dof_specification> : <value>, ...]}
}
```
    * example:
```
ebc_1 = {
    'name' : 'ZeroSurface',
    'region' : 'Surface',
    'dofs' : {'u.all' : 0.0, 'phi.all' : 0.0},
}
```
  * Dirichlet (essential) boundary conditions, short syntax:
```
ebcs = {
    'name' : (<region_name>, {<dof_specification> : <value>[,
                              <dof_specification> : <value>, ...]},...)
}
```
    * example:
```
ebcs = {
    'u1' : ('Left', {'u.all' : 0.0}),
    'u2' : ('Right', {'u.0' : 0.1}),
    'phi' : ('Surface', {'phi.all' : 0.0}),
}
```

## Initial conditions ##

Initial conditions are applied prior to the boundary conditions - no special care must be used for the boundary dofs.

  * long syntax:
```
ic_<number> = {
    'name' : <string>,
    'region' : <region_name>,
    'dofs' : {<dof_specification> : <value>[,
              <dof_specification> : <value>, ...]}
}
```
    * example:
```
ic_1 = {
    'name' : 'ic',
    'region' : 'Omega',
    'dofs' : {'T.0' : 5.0},
}
```
  * short syntax:
```
ics = {
    'name' : (<region_name>, {<dof_specification> : <value>[,
                              <dof_specification> : <value>, ...]},...)
}
```
    * example:
```
ics = {
    'ic' : ('Omega', {'T.0' : 5.0}),
}
```

## Materials ##

Materials are used to define constitutive parameters (e.g. stiffness, permeability, or viscosity), and other non-field arguments of terms (e.g. known traction or volume forces).
Depending on a particular term, the parameters can be constants, functions defined over FE mesh nodes, functions defined in the elements, etc.

  * example:
```
material_10 = {
    'name' : 'm',
    'region' : 'SomeRegion',
    'values' : {
        # This gets tiled to all physical QPs (constant function)
        'val' : [0.0, -1.0, 0.0],
        # This does not - '.' denotes a special value, e.g. a flag.
        '.val0' : [0.0, 0.1, 0.0],
    },
}

material_3 = {
    'name' : 'm',
    'region' : 'SomeRegion',
    'function' : 'some_function',
}

def some_function(ts, coor, region, ig, mode=None):
    out = {}
    if mode == 'qp':
        out['val'] = <array of shape (coor.shape[0], n_row, n_col)>
    else: # special mode
        out['val0'] = True

```

## Equations and terms ##

Equations can be built by combining terms listed in [sfepy\_manual.pdf](http://sfepy.googlecode.com/svn/web/docs/sfepy_manual.pdf).

Examples:

  * Laplace equation:
```
equations = {
    'Temperature' : """dw_laplace.i1.Omega( coef.val, s, t ) = 0"""
}
```
  * Navier-Stokes equations:
```
equations = {
    'balance' :
    """+ dw_div_grad.i2.Omega( fluid.viscosity, v, w )
       + dw_convect.i2.Omega( v, w )
       - dw_grad.i1.Omega( v, r ) = 0""",
    'incompressibility' :
    """dw_div.i1.Omega( q, w ) = 0""",
}
```

## Configuring solvers ##

In SfePy, a non-linear solver has to be specified even when solving a linear problem. The linear problem is/should be then solved in one iteration of the nonlinear solver.

  * linear solver
```
solver_0 = {
    'name' : 'ls',
    'kind' : 'ls.umfpack',
}
```
  * nonlinear solver
```
solver_1 = {
    'name' : 'newton',
    'kind' : 'nls.newton',

    'i_max'      : 1,
    'eps_a'      : 1e-10,
    'eps_r'      : 1.0,
    'macheps'   : 1e-16,
    'lin_red'    : 1e-2, # Linear system error < (eps_a * lin_red).
    'ls_red'     : 0.1,
    'ls_red_warp' : 0.001,
    'ls_on'      : 1.1,
    'ls_min'     : 1e-5,
    'check'     : 0,
    'delta'     : 1e-6,
    'is_plot'    : False,
    'problem'   : 'nonlinear', # 'nonlinear' or 'linear' (ignore i_max)
}
```
  * solver selection
```
options = {
    'nls' : 'newton',
    'ls' : 'ls',
}
```

## Functions ##

Functions are a way of customizing SfePy behavior. They make it possible to define material properties, boundary conditions, parametric sweeps, and other items in an arbitrary manner. Functions are normal Python functions declared in the Problem Definition file, so they can invoke the full power of Python. In order for SfePy to make use of the functions, they must be declared using the function keyword. See below for examples.

```
def get_pars(ts, coors, mode=None, region=None, ig=None, extra_arg=None):
    if mode == 'special':
        if extra_arg == 'hello!':
            ic = 0
        else:
            ic = 1
        return {('x_%s' % ic) : coors[:,ic]}

def get_p_edge(ts, coors, bc=None):
    if bc.name == 'p_left':
        return nm.sin(nm.pi * coors[:,1])
    else:
        return nm.cos(nm.pi * coors[:,1])

def get_circle(coors, domain=None):
    r = nm.sqrt(coors[:,0]**2.0 + coors[:,1]**2.0)
    return nm.where(r < 0.2)[0]

functions = {
    'get_pars1' : (lambda ts, coors, mode=None, region=None, ig=None:
                   get_pars(ts, coors, mode, region, ig, extra_arg='hello!'),),
    'get_p_edge' : (get_p_edge,),
    'get_circle' : (get_circle,),
}

# Just another way of adding a function, besides 'functions' keyword.
function_1 = {
    'name' : 'get_pars2',
    'function' : lambda ts, coors,mode=None,  region=None, ig=None:
        get_pars(ts, coors, mode, region, ig, extra_arg='hi!'),
}
```

## Miscellaneous ##

  * number of elements assembled in one term function call
```
fe = {
    'chunk_size' : 1000
}
```
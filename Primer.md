**NOTE:** The Primer updated for current SfePy is available at http://docs.sfepy.org/doc-devel/primer.html - the text below is the original version that uses some now obsolete syntax and is deprecated.



# Introduction #

Welcome to this primer on SfePy. To get you started, a step-by-step walk-through of the process to solve a simple mechanics problem is presented. The typical process to solve a problem using SfePy is followed: a model is meshed, a problem definition file is drafted, SfePy is run to solve the problem and finally the results of the analysis are illustrated.


## Problem statement ##

A popular test to measure the tensile strength of concrete or asphalt materials is the indirect tensile strength test pictured below.

![http://groups.google.com/group/sfepy-devel/web/ITS.png](http://groups.google.com/group/sfepy-devel/web/ITS.png)

In this test a circular cylindrical specimen is loaded across its diameter to failure. The test is usually done by loading the specimen at a constant deformation rate of 50 mm/minute (say) and measuring the load response. When the tensile stress that develops in the specimen under loading exceeds its tensile strength then the specimen will fail. To model this problem using finite elements the indirect tensile test can be simplified to represent a diametrically point loaded disk as shown below.

![http://groups.google.com/group/sfepy-devel/web/ITS2D.png](http://groups.google.com/group/sfepy-devel/web/ITS2D.png)

The tensile and compressive stresses that develop in the centre of the specimen as a result of the point loads P are a function of the diameter (D) and thickness (t) of the cylindrical specimen. These are, respectively:

<wiki:gadget url="http://mathml-gadget.googlecode.com/svn/trunk/mathml-gadget.xml" border="0" up\_content="sigma\_{t} = (2 cdot P) / (pi cdot t cdot D)"/>

<wiki:gadget url="http://mathml-gadget.googlecode.com/svn/trunk/mathml-gadget.xml" border="0" up\_content="sigma\_{c} = (6 cdot P) / (pi cdot t cdot D)"/>

These solutions may be approximated using finite element analyses. To solve this problem using SfePy the first step is meshing a suitable model.

# Meshing #

Assuming plane strain conditions, the indirect tensile test may be modeled using a 2-D finite element mesh. Furthermore, the geometry of the model is symmetrical about the x- and y-axes passing through the centre of the circle. To take advantage of this symmetry only one quarter of the 2-D model will be meshed and boundary conditions will be established to indicate this symmetry. The meshing program Gmsh is used in this tutorial to very quickly mesh the model.

The indirect tensile test specimen has a diameter of 150 mm. Using Gmsh add three new Points (geometry elementary entities) at the following coordinates:
(0.0 0.0), (75.0,0.0) and (0.0,75.0). Next add two straight lines connecting the points as shown below.

![http://groups.google.com/group/sfepy-devel/web/mesh1.png](http://groups.google.com/group/sfepy-devel/web/mesh1.png)

Next add a Circle arc connecting two of the points to form the quarter circle segment.

![http://groups.google.com/group/sfepy-devel/web/mesh2.png](http://groups.google.com/group/sfepy-devel/web/mesh2.png)

Still under Geometry add a Ruled surface.

![http://groups.google.com/group/sfepy-devel/web/mesh3.png](http://groups.google.com/group/sfepy-devel/web/mesh3.png)


With the geometry of the model defined, add a mesh by clicking on the 2D button under the Mesh functions.

![http://groups.google.com/group/sfepy-devel/web/mesh4.png](http://groups.google.com/group/sfepy-devel/web/mesh4.png)

That's it, we're done with the meshing. Save the mesh in a format that SfePy recognizes. For now use the medit .mesh format e.g. its2D.mesh. (Hint: check the drop down in the _Save As_ dialog for the different formats Gmsh can save to)

If you open the its2D.mesh file using a text editor you'll notice that Gmsh saves the mesh in a 3-D format and includes some extra geometry items that we can delete. I've reformatted the mesh file to a 2-D format and deleted the _Edges_ block. Note that when you do this the file cannot be reopened by Gmsh so it is always a good idea to also save your meshes in Gmsh's native format as well (Shift-Ctrl-S). Click on the following link to download the reformatted mesh file that will be used in the tutorial.

http://groups.google.com/group/sfepy-devel/web/its2D.mesh


You'll notice that the mesh contains 55 vertices (nodes) and 83 triangle elements. The mesh file provides the coordinates of the nodes and the element connectivities. It is important to note that node and element numbering in SfePy start at 0 and not 1 as is the case in Gmsh and other meshing programs. For example, a demo version of medit can be downloaded from the following site:

http://www.ann.jussieu.fr/~frey/software.html

After loading your mesh file with medit you can see the node and element numbering by pressing P and F respectively. The numbering in medit starts at 1 as shown below:

![http://groups.google.com/group/sfepy-devel/web/its2D_4.png](http://groups.google.com/group/sfepy-devel/web/its2D_4.png)

Thus the node at the center of the model in SfePy numbering is 0, and elements 76 and 77 are connected to this node.

Node and element numbers can also be viewed in Gmsh - under the mesh option under the Visibility tab enable the node and surface labels. Note that the surface labels as numbered in Gmsh follow on from the line numbering. So to get the corresponding element number in SfePy you'll need to subtract the number of lines in the Gmsh file + 1. Confused yet? Luckily, SfePy provides some useful mesh functions to indicate which elements are connected to which nodes. Nodes and elements can also be identified by defining regions - but more on this later.

The next step in the process is developing the SfePy problem definition file.

# Problem description #

The programming of the problem description file is well documented in the [SfePy user's guide](http://docs.sfepy.org/doc/users_guide.html). The problem description file used in the tutorial follows:

```
# 04/22/2010 
# Diametrically point loaded 2-D disk.

from sfepy.mechanics.matcoefs import youngpoisson_to_lame

filename = 'its2D.mesh'
output_dir = '/home/grassy/sfepy_dev/sfepy/pyfem/output' # set this to a valid directory you have write access to

young = 2000.0 # Young's modulus [MPa]
poisson = 0.4  # Poisson's ratio

filename_mesh = filename

options = {
    'output_dir' : output_dir,
}

regions = {
    'Omega' : ('all', {}),
    'Left' : ('nodes in (x < 0.001)', {}),
    'Bottom' : ('nodes in (y < 0.001)', {}),
    'Top' : ('node 2', {}),
}

materials = {
    'Asphalt' : ('Omega', {
        'lam' : youngpoisson_to_lame(young, poisson)[0],
        'mu' : youngpoisson_to_lame(young, poisson)[1],
    }),
}

fields = {
    'displacement': ((2,1), 'real', 'Omega', {'Omega' : '2_3_P1'}),
}

integrals = {
    'i1' : ('v', 'gauss_o2_d2'),
}

variables = {
    'u' : ('unknown field', 'displacement', 0),
    'v' : ('test field', 'displacement', 'u'),
}

ebcs = {
    'XSym' : ('Bottom', {'u.1' : 0.0}),
    'YSym' : ('Left', {'u.0' : 0.0}),
    'Load' : ('Top', {'u.0' : 0.0, 'u.1' : -1.0}),
}

equations = {
    'balance_of_forces' :
    """dw_lin_elastic_iso.i1.Omega(Asphalt.lam, Asphalt.mu, v, u ) = 0""",
}

solvers = {
    'ls' : ('ls.scipy_direct', {}),
    'newton' : ('nls.newton', {
        'i_max' : 1,
        'eps_a' : 1e-6,
        'problem' : 'nonlinear'
    }),
}

fe = {
    'chunk_size' : 1000
}
```

It can be downloaded here:

http://groups.google.com/group/sfepy-devel/web/its2D_1.py

Download and open the file in your favorite python editor. Note that you will need to change the location of the output directory to somewhere on your drive. For the analysis we will assume that the material of the test specimen is linear elastic and isotropic. We define two material constants i.e. Young's modulus and Poisson's ratio. The material is assumed to be asphalt concrete having a Young's modulus of 2,000 MPa and a Poisson's ration of 0.4.

Note - be consistent in your choice and use of units. In the tutorial we are using Newton (N), millimeters (mm) and megaPascal (MPa). The [sfepy.mechanics.units module](http://docs.sfepy.org/doc-devel/src/sfepy/mechanics/units.html) might help you in determining which derived units correspond to given basic units.

The following block of code defines regions on your mesh:
```
regions = {
    'Omega' : ('all', {}),
    'Left' : ('nodes in (x < 0.001)', {}),
    'Bottom' : ('nodes in (y < 0.001)', {}),
    'Top' : ('node 2', {}),
}
```

Four regions are defined:
  1. Omega: all the elements in the mesh
  1. Left: the y-axis
  1. Bottom: the x-axis
  1. Top: the topmost node. This is where the load is applied.

Having defined the regions these can be used in other parts of your code. For example, consider the definition of the asphalt material:

```
materials = {
    'Asphalt' : ('Omega', {
        'lam' : youngpoisson_to_lame(young, poisson)[0],
        'mu' : youngpoisson_to_lame(young, poisson)[1],
    }),
}
```

Since the material is defined on Omega it applies to the entire model. We provided the material constants in terms of Young's modulus and Poisson's ratio, but the linear elastic isotropic equation used requires as input Lamé’s parameters. The _youngpoisson\_to\_lame_ function is thus used for conversion. Note that to use this function it was necessary to import the function into the code, which was done up front:

```
from sfepy.mechanics.matcoefs import youngpoisson_to_lame
```

Check out [the sfepy.mechanics.matcoefs module](http://docs.sfepy.org/doc-devel/src/sfepy/mechanics/matcoefs.html) for other useful functions.

The boundary conditions are defined as follows:

```
ebcs = {
    'XSym' : ('Bottom', {'u.1' : 0.0}),
    'YSym' : ('Left', {'u.0' : 0.0}),
    'Load' : ('Top', {'u.0' : 0.0, 'u.1' : -1.0}),
}
```

Now the power of the regions entity becomes apparent. To ensure symmetry about the x-axis, the vertical or y-displacement of the nodes in the _Bottom_ region are prevented or set to zero. Similarly, for symmetry about the y-axis, any horizontal or displacement in the x-direction of the nodes in the _Left_ region or y-axis is prevented. Finally, to indicate the response of the load, the topmost node (number 2) is given a displacement of 1 mm downwards in the vertical or y-direction and displacement of this node in the x-direction is restricted.

That's it - we are now ready to solve the problem.

# Running SfePy #

One option to solve the problem is to run the SfePy _simple.py_ script from the command shell:

```
  $ ./simple.py its2D_1.py
```

For the purpose of this tutorial it is assumed that the problem definition file (its2D\_1.py) is in the same directory as the _simple.py_ script. If you have the _its2D\_1.py_ file in another directory then make sure you include the path to this file as well.

SfePy solves the problem and outputs the solution to the output path (output\_dir) provided in the script. The output file will be in the vtk format by default if this is not explicitly specified and the name of the output file will be the same as that used for the mesh file except with the vtk extension i.e. its2D.vtk.

The vtk format is an ascii format. Open the file using a text editor. You'll notice that the output file includes separate sections:
  * POINTS (these are the model nodes)
  * CELLS (the model element connectivity)
  * VECTORS (the node displacements in the x-, y- and z- directions.

Notice that the y-displacement of node 2 is -1.0 as we set it as a boundary condition.

SfePy includes a script (_postproc.py_) to quickly view the solution. To run this script you need to have [Mayavi](http://code.enthought.com/projects/mayavi/docs/development/html/mayavi/index.html) installed. From the command line issue the following (with the correct paths):

```
  $ ./postproc.py its2D.vtk
```

The solution will be displayed as follows:

![http://groups.google.com/group/sfepy-devel/web/its2D_1.png](http://groups.google.com/group/sfepy-devel/web/its2D_1.png)

The figure shows the average displacements in the model but we are more interested in the stresses. To get these we need to modify the problem description file.

# Post-processing #

SfePy provides functions to calculate stresses and strains. We'll include a function to calculate these and update the problem material definition and options to call this function as a _post\_process\_hook_. Save this file as [its2D\_2.py](http://groups.google.com/group/sfepy-devel/web/its2D_2.py).

```
from its2D_1 import *

from sfepy.mechanics.matcoefs import stiffness_tensor_youngpoisson

def stress_strain(out, pb, state, extend = False):
    '''
    Calculate and output strain and stress for given displacements.
    '''
    from sfepy.base.base import Struct
    from sfepy.fem.evaluate import eval_term_op

    strain = eval_term_op(state, 'de_cauchy_strain.i1.Omega(u)', pb)
    stress = eval_term_op(state, 'de_cauchy_stress.i1.Omega(Asphalt.D,u)', pb)

    out['cauchy_strain'] = Struct(name = 'output_data', mode = 'cell', data = strain, dofs = None)
    out['cauchy_stress'] = Struct(name = 'output_data', mode = 'cell', data = stress, dofs = None)

    return out

materials['Asphalt'][1].update({'D' : stiffness_tensor_youngpoisson(2, young, poisson)})

options.update({'post_process_hook' : 'stress_strain',})
```

The updated file imports all of the previous definitions in _its2D\_1.py_. The stress function (_de\_cauchy\_stress_) requires as input the stiffness tensor - thus it was necessary to update the materials accordingly. The problem options were also updated to call the stress\_strain function as a _post\_process\_hook_.

Run SfePy to solve the updated problem and view the solution (assuring the correct paths):

```
  $ ./simple.py its2D_2.py   
  $ ./postproc.py its2D.vtk
```

In addition to the node displacements, the vtk output now also includes the stresses and strains averaged in the elements:

![http://groups.google.com/group/sfepy-devel/web/its2D_2.png](http://groups.google.com/group/sfepy-devel/web/its2D_2.png)

Remember the objective was to determine the stresses at the centre of the specimen under a load P. The solution as currently derived is expressed in terms of a global displacement vector (u). The global (residual) force vector (f) is a function of the global displacement vector and the global stiffness matrix (K) as: f = Ku.

In addition to solving problems using the _simple.py_ script you can also run SfePy interactively. This requires that [ipython](http://ipython.scipy.org/moin/) be installed. To run SfePy interactively, run the following command:

```
  $ ./isfepy
```

Once isfepy loads up, issue the following command:

```
In [1]: pb,vec,data=pde_solve('its2D_2.py')
```

The problem is solved and the problem definition and solution are provided in the _pb_ and _vec_ variables - see for yourself:

```
In [2]: pb
Out[2]: ProblemDefinition

In [3]: vec
Out[3]: 
array([ 0.        ,  0.        ,  0.22608933, ..., -0.12051821,
        0.05335311, -0.0677574 ])

```

_vec_ is a [NumPy](http://numpy.scipy.org/) array. If an array is too large to be printed then NumPy automatically skips the central part of the array and only prints the corners. To disable this behaviour and force NumPy to print the entire array, you can change the printing options using _set\_printoptions_:

```
In [4]: nm.set_printoptions(threshold=nm.nan)

In [5]: vec
Out[5]: 
array([  0.00000000e+00,   0.00000000e+00,   2.26089335e-01,
         0.00000000e+00,   0.00000000e+00,  -1.00000000e+00,
         6.63971215e-02,   0.00000000e+00,   1.24441304e-01,
         0.00000000e+00,   1.70385399e-01,   0.00000000e+00,
         1.99298768e-01,   0.00000000e+00,   2.18582169e-01,
         0.00000000e+00,   2.25796244e-01,   0.00000000e+00,
         0.00000000e+00,  -8.79805516e-02,   0.00000000e+00,
        -1.82185927e-01,   0.00000000e+00,  -2.80990423e-01,
         0.00000000e+00,  -4.00595135e-01,   0.00000000e+00,
        -5.50110472e-01,   0.00000000e+00,  -7.45248105e-01,
         2.27070400e-01,  -2.04441298e-04,   2.20041801e-01,
        -3.26787845e-03,   2.05201142e-01,  -1.09412111e-02,
         1.80098205e-01,  -2.65392990e-02,   1.55431639e-01,
        -4.82843331e-02,   1.28652055e-01,  -7.93731154e-02,
         9.21889931e-02,  -1.32683205e-01,   6.48084676e-02,
        -2.15748378e-01,   5.64208432e-02,  -3.47335767e-01,
         6.06872070e-02,  -5.38055540e-01,   1.09511997e-01,
        -3.36581071e-01,   1.40791478e-01,  -1.43717901e-01,
         1.65544415e-01,  -7.99009303e-02,   7.60332236e-02,
        -3.05735492e-01,   1.94131192e-01,  -3.14738683e-02,
         1.86892641e-01,  -7.27770330e-02,   8.91897463e-02,
        -1.91994617e-01,   1.54623330e-01,  -4.60437943e-02,
         1.20254408e-01,  -1.12774894e-01,   1.09504925e-01,
        -4.70318595e-02,   1.77917973e-01,  -1.08193881e-01,
         1.83995155e-01,  -5.65245956e-02,   1.57499020e-01,
        -1.70118441e-01,   1.69618983e-01,  -8.18631674e-02,
         1.22044326e-01,  -2.11011757e-01,   2.16704241e-01,
        -2.09744554e-02,   2.13139411e-01,  -1.95199341e-02,
         7.60146118e-02,  -4.95813366e-01,   2.00891096e-01,
        -5.18626156e-02,   2.24565301e-01,  -7.60322658e-03,
         1.31618167e-01,  -2.57287456e-01,   7.84055846e-02,
        -3.93575270e-01,   2.01032828e-01,  -4.07244993e-02,
         1.51756670e-01,  -1.71585144e-01,   9.14563247e-02,
        -2.76913189e-01,   1.30598428e-01,  -2.24890151e-01,
         1.39613746e-01,  -1.35535326e-01,   5.27252944e-02,
        -1.39543341e-01,   1.67805867e-01,  -1.20518214e-01,
         5.33531054e-02,  -6.77573971e-02])

In [6]: vec.shape
Out[6]: (110,)
```

So _vec_, the global displacement vector, holds the x- and y-displacements at the 55 nodes in the model.

The global stiffness matrix is saved in _pb_ as a [sparse](http://docs.scipy.org/doc/scipy/reference/sparse.html) matrix:

```
In [7]: print pb.mtx_a
------> print(pb.mtx_a)
  (0, 0)	2443.95959851
  (0, 6)	-2110.99917491
  (0, 13)	-332.960423597
  (0, 14)	1428.57142857
  (1, 1)	4048.78343529
  (1, 2)	-1354.87004384
  (1, 51)	-609.367453538
  (1, 52)	-1869.0018791
  (1, 91)	-357.41672785
  (1, 92)	1510.24654193
  (2, 1)	-1354.87004384
  (2, 2)	4121.03202907
  (2, 3)	-1696.54911732
  (2, 47)	76.2400806561
  (2, 48)	-1669.59247304
  (2, 51)	-1145.85294856
  (2, 52)	2062.13955556
  (3, 2)	-1696.54911732
  (3, 3)	4410.17902905
  (3, 4)	-1872.87344838
  (3, 41)	-130.515009576
  (3, 42)	-1737.33263802
  (3, 47)	-710.241453776
  (3, 48)	1880.20135513
  (4, 3)	-1872.87344838
  :	:
  (90, 80)	-1610.0550578
  (90, 85)	-199.343680224
  (90, 86)	-2330.41406097
  (90, 89)	-575.80373408
  (90, 90)	7853.23899229
  (91, 1)	-357.41672785
  (91, 7)	1735.59411191
  (91, 49)	-464.976034459
  (91, 50)	-1761.31189004
  (91, 51)	-3300.45367361
  (91, 52)	1574.59387937
  (91, 87)	-250.325600254
  (91, 88)	1334.11823335
  (91, 91)	9219.18643706
  (91, 92)	-2607.52659081
  (92, 1)	1510.24654193
  (92, 7)	-657.361661955
  (92, 49)	-1761.31189004
  (92, 50)	54.1134516246
  (92, 51)	1574.59387937
  (92, 52)	-315.793227627
  (92, 87)	1334.11823335
  (92, 88)	-4348.13351285
  (92, 91)	-2607.52659081
  (92, 92)	9821.16012014

In [8]: pb.mtx_a.shape
Out[8]: (93, 93)
```

One would expect the shape of the global stiffness matrix to be (110,110) i.e. to have the same number of rows and columns as _vec_. This matrix has in fact been reduced by the **fixed** degrees of freedom imposed by the boundary conditions set at the nodes on symmetry axes. To _restore_ the matrix, temporarily remove the imposed boundary conditions:

```
In [9]: pb.time_update(conf_ebc={},conf_epbc={},conf_lcbc={})
sfepy: updating materials...
sfepy:       Asphalt
sfepy:   ...done in 0.01 s
sfepy: updating variables...
sfepy:   ...done
```

The residual force vector (f) can now be determined:

```
In [10]: f=pb.evaluator.eval_residual(vec)

In [11]: f
Out[11]: 
array([ -2.86489070e+01,   8.63500981e+01,   5.70847704e-14,
         3.77394659e-01,   3.75260226e+02,  -6.04894252e+02,
        -1.13686838e-13,   1.59660295e+02,  -4.61852778e-14,
         1.23660375e+02,  -3.50830476e-13,   1.00377488e+02,
        -8.57092175e-14,   6.83165492e+01,  -1.00364161e-13,
         4.71094090e+01,   5.97855099e-14,   1.90426427e+01,
        -5.42687026e+01,   8.70414851e-14,  -4.58288484e+01,
        -1.98951966e-13,  -5.16127155e+01,  -3.10862447e-13,
        -5.46877815e+01,  -1.13686838e-13,  -6.77980869e+01,
        -4.12114787e-13,  -7.24151845e+01,   7.95807864e-13,
        -1.65645275e-13,   1.16739951e-13,   3.99680289e-13,
         4.24937863e-14,   3.02258218e-13,   5.10702591e-15,
         8.43491943e-14,   4.50160742e-14,  -3.21964677e-13,
         6.12843110e-14,  -2.73114864e-14,  -4.08562073e-14,
         1.59872116e-13,  -1.77635684e-13,   6.03961325e-14,
         8.52651283e-14,  -1.98951966e-13,  -1.13686838e-13,
        -1.70530257e-13,   1.42108547e-13,   4.19220214e-13,
         3.12638804e-13,   8.17124146e-14,   1.52766688e-13,
        -1.45661261e-13,   1.27897692e-13,  -3.41060513e-13,
        -5.68434189e-14,  -1.84741111e-13,  -7.10542736e-14,
        -2.28705943e-13,   3.55271368e-14,   1.31450406e-13,
         3.41060513e-13,   1.24344979e-14,   4.26325641e-14,
         2.38031816e-13,  -3.48165941e-13,   8.17124146e-14,
         3.41060513e-13,  -1.10134124e-13,   3.97903932e-13,
        -2.77111667e-13,  -4.44089210e-14,   7.65609798e-13,
         1.63424829e-13,   2.98427949e-13,  -4.05009359e-13,
        -1.54543045e-13,   2.77111667e-13,   6.03961325e-14,
        -3.53050922e-14,  -7.10542736e-15,   2.84217094e-14,
         6.32383035e-13,  -6.39488462e-13,   1.43884904e-13,
        -2.24820162e-13,   5.09814413e-13,  -2.13162821e-13,
         7.28306304e-14,  -2.09610107e-13,   5.68434189e-14,
        -2.84217094e-14,  -6.57252031e-14,  -1.81188398e-13,
         3.76587650e-13,  -2.27373675e-13,  -1.24344979e-13,
        -7.81597009e-14,   4.74287276e-13,  -2.07833750e-13,
         1.06581410e-13,  -6.03961325e-14,  -3.23296945e-13,
        -2.27373675e-13,  -7.81597009e-14,   2.38031816e-13,
         1.42108547e-13,   5.68434189e-14])

In [12]: f.shape
Out[12]: (110,)
```

Remember to restore the original boundary conditions:

```
In [13]: pb.time_update()
```

To view the residual force vector, we can save it to a vtk file:

```
In [14]: pb.save_state('file.vtk',f)
```

Running the _postproc.py_ script on _file.vtk_ shows the average nodal forces:

![http://groups.google.com/group/sfepy-devel/web/its2D_3.png](http://groups.google.com/group/sfepy-devel/web/its2D_3.png)

Let's change the shape of _f_ to reflect the forces at the nodes:

```
In [15]: f.shape = (55,2)

In [16]: f
Out[16]: 
array([[ -2.86489070e+01,   8.63500981e+01],
       [  5.70847704e-14,   3.77394659e-01],
       [  3.75260226e+02,  -6.04894252e+02],
       [ -1.13686838e-13,   1.59660295e+02],
       [ -4.61852778e-14,   1.23660375e+02],
       [ -3.50830476e-13,   1.00377488e+02],
       [ -8.57092175e-14,   6.83165492e+01],
       [ -1.00364161e-13,   4.71094090e+01],
       [  5.97855099e-14,   1.90426427e+01],
       [ -5.42687026e+01,   8.70414851e-14],
       [ -4.58288484e+01,  -1.98951966e-13],
       [ -5.16127155e+01,  -3.10862447e-13],
       [ -5.46877815e+01,  -1.13686838e-13],
       [ -6.77980869e+01,  -4.12114787e-13],
       [ -7.24151845e+01,   7.95807864e-13],
       [ -1.65645275e-13,   1.16739951e-13],
       [  3.99680289e-13,   4.24937863e-14],
       [  3.02258218e-13,   5.10702591e-15],
       [  8.43491943e-14,   4.50160742e-14],
       [ -3.21964677e-13,   6.12843110e-14],
       [ -2.73114864e-14,  -4.08562073e-14],
       [  1.59872116e-13,  -1.77635684e-13],
       [  6.03961325e-14,   8.52651283e-14],
       [ -1.98951966e-13,  -1.13686838e-13],
       [ -1.70530257e-13,   1.42108547e-13],
       [  4.19220214e-13,   3.12638804e-13],
       [  8.17124146e-14,   1.52766688e-13],
       [ -1.45661261e-13,   1.27897692e-13],
       [ -3.41060513e-13,  -5.68434189e-14],
       [ -1.84741111e-13,  -7.10542736e-14],
       [ -2.28705943e-13,   3.55271368e-14],
       [  1.31450406e-13,   3.41060513e-13],
       [  1.24344979e-14,   4.26325641e-14],
       [  2.38031816e-13,  -3.48165941e-13],
       [  8.17124146e-14,   3.41060513e-13],
       [ -1.10134124e-13,   3.97903932e-13],
       [ -2.77111667e-13,  -4.44089210e-14],
       [  7.65609798e-13,   1.63424829e-13],
       [  2.98427949e-13,  -4.05009359e-13],
       [ -1.54543045e-13,   2.77111667e-13],
       [  6.03961325e-14,  -3.53050922e-14],
       [ -7.10542736e-15,   2.84217094e-14],
       [  6.32383035e-13,  -6.39488462e-13],
       [  1.43884904e-13,  -2.24820162e-13],
       [  5.09814413e-13,  -2.13162821e-13],
       [  7.28306304e-14,  -2.09610107e-13],
       [  5.68434189e-14,  -2.84217094e-14],
       [ -6.57252031e-14,  -1.81188398e-13],
       [  3.76587650e-13,  -2.27373675e-13],
       [ -1.24344979e-13,  -7.81597009e-14],
       [  4.74287276e-13,  -2.07833750e-13],
       [  1.06581410e-13,  -6.03961325e-14],
       [ -3.23296945e-13,  -2.27373675e-13],
       [ -7.81597009e-14,   2.38031816e-13],
       [  1.42108547e-13,   5.68434189e-14]])
```

The force in the x- and y-directions at node 2 is therefore:

```
In [17]: f[2]
Out[17]: array([ 375.26022639, -604.89425239])
```

Great, we have the vertical load or force apparent at node 2 i.e. 604.894 Newton. Since we modeled the problem using **symmetry**, the actual load applied to achieve the nodal displacement of 1 mm is 2 x 604.894 = 1209.7885 N. Applying the indirect tensile strength stress equation, the horizontal tensile stress at the center of the specimen per unit thickness is 5.1345 MPa/mm and the vertical compressive stress per unit thickness is 15.4035 MPa/mm. The per unit thickness results are in terms of the plane strain conditions assumed for the 2D model.

Previously we had calculated the stresses in the model but these were averaged from those calculated at gauss quadrature points within the elements. It is possible to provide custom integrals to allow the calculation of stresses with the gauss quadrature points at the element nodes. This will provide us a more accurate estimate of the stress at the centre of the specimen located at node 0. The code below outlines one way to achieve this.

This code can be downloaded [here](http://groups.google.com/group/sfepy-devel/web/its2D_3.py).

```
from its2D_1 import *
import numpy as np

from sfepy.mechanics.matcoefs import stiffness_tensor_youngpoisson
from sfepy.fem.geometry_element import geometry_data

gdata = geometry_data['2_3']
nc = len(gdata.coors)

def area_tri(pb): 
    """
    Calculates area of triangle elements 
    """
    areas = []
    tri_xy = []
    conns = pb.domain.mesh.conns[0]
    coors = pb.domain.mesh.coors
    for e in conns:
        nc = []
        for n in e:
            nc.append(coors[n].tolist())
        tri_xy.append(nc)
    for en in tri_xy:
        areas.append(0.5*abs(en[0][0]*en[1][1]-en[0][0]*en[2][1]+en[1][0]*en[2][1]-en[1][0]*en[0][1]+en[2][0]*en[0][1]-en[2][0]*en[1][1]))

    return areas

def nodal_stress(out, pb, state, extend = False):
    '''
    Calculate stresses at nodes.
    '''
    from sfepy.fem.evaluate import eval_term_op
    from sfepy.fem.mesh import make_inverse_connectivity

    stress = eval_term_op(state, 'dq_cauchy_stress.ivn.Omega(Asphalt.D, u)', pb)
    n_els, iconn =  make_inverse_connectivity(pb.domain.mesh.conns, pb.domain.mesh.n_nod, ret_offsets=False)

    pb.time_update(conf_ebc={},conf_epbc={},conf_lcbc={})
    f=pb.evaluator.eval_residual(state)
    pb.time_update()
    f.shape = (pb.domain.mesh.n_nod,2)

    P=2.*f[2][1]

    areas = area_tri(pb) 

    j = 0
    ta = 0. # Total area

    for i in range(n_els[0]): # find all the elements connected to node 0 (centre of model)
        j = j+i+1
        a=areas[iconn[j]]
        ta += a
        if i==0:
            s=stress[iconn[j]]*a
        else:
            s+=stress[iconn[j]]*a
    s/=ta

    print '=================================================================='
    print 'Load to give 1 mm displacement = %s Newton ' % (-P)

    print 'Analytical solution'
    print '==================='
    print 'Horizontal tensile stress = %s MPa/mm' % (-2.*P/(np.pi*150.)) 
    print 'Vertical compressive stress = %s MPa/mm' % (-6.*P/(np.pi*150.)) 

    print 'FEM solution'
    print '============'
    print 'Horizontal tensile stress = %s MPa/mm' % s[0][0][0]
    print 'Vertical compressive stress = %s MPa/mm' % -s[0][1][0]

    return out

materials['Asphalt'][1].update({'D' : stiffness_tensor_youngpoisson(2, young, poisson)})
integrals.update({'ivn' : ('v', 'custom', gdata.coors, [gdata.volume / nc] * nc),})

options.update({'post_process_hook' : 'nodal_stress',})
```

An interesting function to retrieve information about nodes and the elements connected to these nodes is the _make\_inverse\_connectivity_ function:
```
n_els, iconn =  make_inverse_connectivity(pb.domain.mesh.conns, pb.domain.mesh.n_nod, ret_offsets=False)
```

It requires as input the mesh connectivity and the number of nodes in the mesh. The output is easily understood when run through _isfepy_:
```
In [1]: pb,vec,data=pde_solve('/home/grassy/its2D_3.py')
In [2]: nm.set_printoptions(threshold=nm.nan)
In [3]: from sfepy.fem.mesh import make_inverse_connectivity
In [4]: n_els, iconn =  make_inverse_connectivity(pb.domain.mesh.conns, pb.domain.mesh.n_nod, ret_offsets=False)
In [5]: n_els
Out[5]: 
array([2, 1, 1, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3,
       4, 3, 6, 6, 6, 6, 6, 6, 7, 6, 7, 5, 6, 5, 6, 6, 5, 6, 6, 6, 7, 6, 7,
       6, 6, 5, 4, 7, 6, 5, 6, 6])

In [6]: iconn
Out[6]: 
array([ 0, 76,  0, 77,  0, 42,  0, 44,  0, 77,  0, 79,  0, 80,  0,  3,  0,
        5,  0, 80,  0,  1,  0,  5,  0,  7,  0,  1,  0, 29,  0, 30,  0, 17,
        0, 29,  0, 34,  0, 17,  0, 41,  0, 42,  0, 71,  0, 76,  0, 78,  0,
        4,  0, 71,  0, 72,  0,  0,  0,  4,  0,  8,  0,  0,  0, 31,  0, 32,
        0, 15,  0, 31,  0, 35,  0, 15,  0, 43,  0, 44,  0, 41,  0, 42,  0,
       48,  0, 48,  0, 49,  0, 51,  0, 47,  0, 50,  0, 51,  0,  9,  0, 46,
        0, 47,  0,  9,  0, 28,  0, 52,  0, 52,  0, 58,  0, 68,  0, 68,  0,
       74,  0, 75,  0, 54,  0, 70,  0, 74,  0, 54,  0, 55,  0, 56,  0, 57,
        0, 43,  0, 44,  0, 56,  0, 39,  0, 40,  0, 55,  0, 57,  0, 65,  0,
       66,  0,  2,  0, 11,  0, 12,  0, 14,  0, 18,  0, 20,  0, 26,  0, 28,
        0, 52,  0, 58,  0, 59,  0, 61,  0,  0,  0,  8,  0, 13,  0, 23,  0,
       32,  0, 38,  0,  1,  0,  7,  0, 10,  0, 21,  0, 25,  0, 30,  0, 16,
        0, 26,  0, 36,  0, 45,  0, 59,  0, 60,  0,  2,  0,  4,  0,  8,  0,
       12,  0, 13,  0, 72,  0, 73,  0,  3,  0,  5,  0,  6,  0,  7,  0, 10,
        0, 19,  0,  2,  0,  6,  0, 18,  0, 19,  0, 73,  0, 81,  0, 82,  0,
        3,  0,  6,  0, 79,  0, 80,  0, 81,  0, 11,  0, 16,  0, 20,  0, 22,
        0, 60,  0, 63,  0,  9,  0, 26,  0, 28,  0, 45,  0, 46,  0, 14,  0,
       20,  0, 24,  0, 53,  0, 63,  0, 64,  0, 10,  0, 11,  0, 18,  0, 19,
        0, 21,  0, 22,  0, 12,  0, 13,  0, 14,  0, 23,  0, 24,  0, 27,  0,
       33,  0, 37,  0, 49,  0, 50,  0, 51,  0, 25,  0, 27,  0, 29,  0, 30,
        0, 33,  0, 34,  0, 15,  0, 35,  0, 40,  0, 43,  0, 55,  0, 56,  0,
       16,  0, 21,  0, 22,  0, 25,  0, 27,  0, 36,  0, 37,  0, 17,  0, 33,
        0, 34,  0, 41,  0, 48,  0, 49,  0, 23,  0, 24,  0, 38,  0, 39,  0,
       53,  0, 66,  0, 67,  0, 31,  0, 32,  0, 35,  0, 38,  0, 39,  0, 40,
        0, 36,  0, 37,  0, 45,  0, 46,  0, 47,  0, 50,  0, 53,  0, 62,  0,
       64,  0, 67,  0, 69,  0, 54,  0, 57,  0, 65,  0, 70,  0, 65,  0, 66,
        0, 67,  0, 69,  0, 70,  0, 74,  0, 75,  0, 58,  0, 61,  0, 62,  0,
       68,  0, 69,  0, 75,  0, 71,  0, 72,  0, 73,  0, 78,  0, 82,  0, 59,
        0, 60,  0, 61,  0, 62,  0, 63,  0, 64,  0, 76,  0, 77,  0, 78,  0,
       79,  0, 81,  0, 82])

In [7]: n_els, iconn =  make_inverse_connectivity(pb.domain.mesh.conns, pb.domain.mesh.n_nod, ret_offsets=True)

In [8]: n_els
Out[8]: 
array([  0,   2,   3,   4,   7,  10,  13,  16,  19,  22,  25,  28,  31,
        34,  37,  40,  43,  46,  49,  52,  55,  58,  61,  64,  68,  71,
        77,  83,  89,  95, 101, 107, 114, 120, 127, 132, 138, 143, 149,
       155, 160, 166, 172, 178, 185, 191, 198, 204, 210, 215, 219, 226,
       232, 237, 243, 249])
```

As you can see _n\_els_ provides the number of elements connected to each of the nodes i.e. node 0 has 2, node 3 has 3, etc. Some of the nodes have 7 elements connected! If the _ret\_offsets_ boolean is set to True then _n\_els_ returns the cumulative number of elements offset from the previous. The _iconn_ array indicates which elements are connected, separated by 0. The zeros in _iconn_ correspond to element groups - an element is referred to by (ig, iel) - the group number and its index within a group. You get different groups either when having both triangles and rectangles in one mesh, and/or by using several element groups ids in the mesh file.

Note that the contribution of the nodes have been weighted against the areas of the connected elements.

Running _its2D\_3.py_ through SfePy provides the following solution:
```
Load to give 1 mm displacement = 1209.78850478 Newton 

Analytical solution
===================
Horizontal tensile stress = 5.1345018835 MPa/mm
Vertical compressive stress = 15.4035056505 MPa/mm
FEM solution
============
Horizontal tensile stress = 4.58038100514 MPa/mm
Vertical compressive stress = 15.6461807296 MPa/mm
```

Not bad for such a coarse mesh. Re-running the problem using a [finer mesh](http://sfepy-devel.googlegroups.com/web/big.mesh) gives the following solution:

```
Load to give 1 mm displacement = 740.778789327 Newton 

Analytical solution
===================
Horizontal tensile stress = 3.14396282824 MPa/mm
Vertical compressive stress = 9.43188848472 MPa/mm
FEM solution
============
Horizontal tensile stress = 3.14846573813 MPa/mm
Vertical compressive stress = 9.41900696084 MPa/mm
```

It appears as if the analytical solution for the horizontal tensile stress under a 1 mm displacement is tending towards pi.

To wrap this tutorial up let's explore SfePy's probing functions.

# Probing #

As a bonus for sticking to the end of this tutorial see the following problem definition file that provides SfePy functions to quickly and neatly _probe_ the solution.

```
from its2D_1 import *

from sfepy.mechanics.matcoefs import stiffness_tensor_youngpoisson

def stress_strain(out, pb, state, extend = False):
    from sfepy.base.base import Struct
    from sfepy.fem.evaluate import eval_term_op

    strain = eval_term_op(state, 'de_cauchy_strain.i1.Omega(u)', pb)
    stress = eval_term_op(state, 'de_cauchy_stress.i1.Omega(Asphalt.D,u)', pb)
    out['cauchy_strain'] = Struct(name = 'output_data', mode = 'cell', data = strain, dofs = None)
    out['cauchy_stress'] = Struct(name = 'output_data', mode = 'cell', data = stress, dofs = None)

    return out

def gen_lines(problem):
    from sfepy.fem.probes import LineProbe
    mesh = problem.domain.mesh
    ps0 = [[0.0,  0.0],[ 0.0,  0.0]]
    ps1 = [[75.0, 0.0],[ 0.0, 75.0]]

    # Use adaptive probe with 10 inital points.
    n_point = -10

    labels = ['%s -> %s' % (p0, p1) for p0, p1 in zip(ps0, ps1)]
    probes = []
    for ip in xrange(len(ps0)):
        p0, p1 = ps0[ip], ps1[ip]
        probes.append(LineProbe(p0, p1, n_point, mesh))

    return probes, labels


def probe_hook(data, probe, label, problem):
    import matplotlib.pyplot as plt
    import matplotlib.font_manager as fm

    def get_it(name, var_name):
        var = problem.variables[var_name]
        if name == var_name:
            indx = problem.variables.di.indx[var.name]
            var.data_from_state(data[name].data, indx)
        else:
            var.data_from_data(data[name].data)
        pars, vals = probe(var)
        vals = vals.squeeze()

        return pars, vals

    results = {}
    results['u'] = get_it('u', 'u')
    results['cauchy_strain'] = get_it('cauchy_strain', 's')
    results['cauchy_stress'] = get_it('cauchy_stress', 's')
    fig = plt.figure()
    plt.clf()
    fig.subplots_adjust(hspace=0.4)
    plt.subplot(311)
    pars, vals = results['u']
    for ic in range(vals.shape[1]):
        plt.plot(pars, vals[:,ic], label=r'$u_{%d}$' % (ic + 1),
                 lw=1, ls='-', marker='+', ms=3)
    plt.ylabel('displacements')
    plt.xlabel('probe %s' % label, fontsize=8)
    plt.legend(loc='best', prop=fm.FontProperties(size=10))

    sym_indices = ['11', '22', '12']

    plt.subplot(312)
    pars, vals = results['cauchy_strain']
    for ic in range(vals.shape[1]):
        plt.plot(pars, vals[:,ic], label=r'$e_{%s}$' % sym_indices[ic],
                 lw=1, ls='-', marker='+', ms=3)
    plt.ylabel('Cauchy strain')
    plt.xlabel('probe %s' % label, fontsize=8)
    plt.legend(loc='best', prop=fm.FontProperties(size=8))

    plt.subplot(313)
    pars, vals = results['cauchy_stress']
    for ic in range(vals.shape[1]):
        plt.plot(pars, vals[:,ic], label=r'$\sigma_{%s}$' % sym_indices[ic],
                 lw=1, ls='-', marker='+', ms=3)
    plt.ylabel('Cauchy stress')
    plt.xlabel('probe %s' % label, fontsize=8)
    plt.legend(loc='best', prop=fm.FontProperties(size=8))

    return plt.gcf(), results

materials['Asphalt'][1].update({'D' : stiffness_tensor_youngpoisson(2, young, poisson)})
fields.update({'sym_tensor'  : ((3,1), 'real', 'Omega', {'Omega' : '2_3_P0'}),})
variables.update({'s' : ('parameter field', 'sym_tensor', None),})    

options.update({
    'output_format'     : 'h5', # VTK reader cannot read cell data yet for probing
    'post_process_hook' : 'stress_strain',
    'gen_probes'        : 'gen_lines',
    'probe_hook'        : 'probe_hook',
})
```

http://groups.google.com/group/sfepy-devel/web/its2D_4.py

Probing applies interpolation to output the solution along specified paths. For the tutorial, line probing is done along the x- and y-axes of the model.

Notice that the output\_format has been defined as _h5_. To apply probing first solve the problem as usual:

```
  $ ./simple.py its2D_4.py
```

This will write the solution to the output directory indicated. Then run the SfePy _probe.py_ script on the solution:

```
  $ ./probe.py its2D_4.py its2D.h5
```

The results of the probing will be written to text files and the following figures will be generated. These figures show the displacements, normal stresses and strains as well as shear stresses and strains along the probe paths. Note that you need [matplotlib](http://matplotlib.sourceforge.net/) installed to run this script.

![http://groups.google.com/group/sfepy-devel/web/its2D_0.png](http://groups.google.com/group/sfepy-devel/web/its2D_0.png)

![http://groups.google.com/group/sfepy-devel/web/its2D_01.png](http://groups.google.com/group/sfepy-devel/web/its2D_01.png)

# Acknowledgements #

Finally, a shout out to all the members of SfePy and especially Robert and Logan for their assistance - guys, thanks for your patience! I have tried to capture in this tutorial most of what I learnt during the first few weeks after being introduced to SfePy. I continue to learn everyday and realize that I've only scratched the surface. In the spirit of SfePy, I encourage you to contribute to the project. Please edit this tutorial as you see fit.

Andre Smit <br>
Austin, TX <br>
27 April 2010 (original posting)<br>
<br>
<b>Notes:</b>

<ol><li>One could alternatively use pb.solve() in place of eval_term_op() in stress_strain().<br>
</li><li>It is advisable to weight the contributions of each node in nodal_stress() by the areas (volumes in 3D) of the elements it is in. In fact, Variable.data_from_qp() would do that for you! Yes, another undocumented function :].</li></ol>



# The Input File: parameters.in {#input_file}

Most PRISMS-PF users will spend the majority of their time interacting with the input file, parameters.h. This file allows users to specify the computational domain, the mesh, the time step parameters, the boundary conditions, the output, and model constants (and more). This file is read as a text file, so modifications to it do not require recompilation of the application.

Here is an example of an input file from the [allenCahn application](https://github.com/prisms-center/phaseField/blob/master/applications/allenCahn/parameters.in):
```
# Parameter list for the Allen-Cahn example application
# Refer to the PRISMS-PF manual for use of these parameters in the source code.

# =================================================================================
# Set the number of dimensions (2 or 3 for a 2D or 3D calculation)
# =================================================================================
set Number of dimensions = 2

# =================================================================================
# Set the length of the domain in all three dimensions
# (Domain size Z ignored in 2D)
# =================================================================================
# Each axes spans from zero to the specified length
set Domain size X = 100
set Domain size Y = 100
set Domain size Z = 100

# =================================================================================
# Set the element parameters
# =================================================================================
# The number of elements in each direction is 2^(refineFactor) * subdivisions
# Subdivisions Z ignored in 2D
# For optimal performance, use refineFactor primarily to determine the element size
set Subdivisions X = 1
set Subdivisions Y = 1
set Subdivisions Z = 1

set Refine factor = 8

# Set the polynomial degree of the element (allowed values: 1, 2, or 3)
set Element degree = 1

# =================================================================================
# Set the time step parameters
# =================================================================================
# The size of the time step
set Time step = 1.0e-2

# The simulation ends when either the number of time steps is reached or the
# simulation time is reached.
set Number of time steps = 5000

# =================================================================================
# Set the output parameters
# =================================================================================
# Type of spacing between outputs ("EQUAL_SPACING", "LOG_SPACING", "N_PER_DECADE",
# or "LIST")
set Output condition = EQUAL_SPACING

# Number of times the program outputs the fields (total number for "EQUAL_SPACING"
# and "LOG_SPACING", number per decade for "N_PER_DECADE", ignored for "LIST")
set Number of outputs = 5

# The number of time steps between updates being printed to the screen
set Skip print steps = 1000

# =================================================================================
# Set the boundary conditions
# =================================================================================
# Set the boundary condition for each variable, where each variable is given by
# its name, as defined in equations.h. The four boundary condition
# types are NATURAL, DIRICHLET, NON_UNIFORM_DIRICHLET and PERIODIC. If all
# of the boundaries have the same boundary condition, only one boundary condition
# type needs to be given. If multiple boundary condition types are needed, give a
# comma-separated list of the types. The order is the miniumum of x, maximum of x,
# minimum of y, maximum of y, minimum of z, maximum of z (i.e left, right, bottom,
# top in 2D and left, right, bottom, top, front, back in 3D). The value of a
# Dirichlet BC is specfied in the following way -- DIRCHILET: val -- where 'val' is
# the desired value. If the boundary condition is NON_UNIFORM_DIRICHLET, the
# boundary condition should be specified in the appropriate function in 'ICs_and_BCs.h'.
# Example 1: All periodic BCs for variable 'c'
# set Boundary condition for variable c = PERIODIC
# Example 2: Zero-derivative BCs on the left and right, Dirichlet BCs with value
# 1.5 on the top and bottom for variable 'n' in 2D
# set Boundary condition for variable n = NATURAL, NATURAL, DIRICHLET: 1.5, DIRICHLET: 1.5

set Boundary condition for variable n = NATURAL

# =================================================================================
# Set the model constants
# =================================================================================
# Set the user-defined model constants, which must have a counter-part given in
# customPDE.h. These are most often used in the residual equations in equations.h,
# but may also be used for initial conditions and nucleation calculations. The type
# options currently are DOUBLE, INT, BOOL, TENSOR, and [symmetry] ELASTIC CONSTANTS
# where [symmetry] is ISOTROPIC, TRANSVERSE, ORTHOTROPIC, or ANISOTROPIC.

# The mobility, MnV in equations.h
set Model constant MnV = 1.0, DOUBLE

# The gradient energy coefficient, KnV in equations.h
set Model constant KnV = 2.0, DOUBLE
```

The syntax for setting each input parameter is:
```
set [parameter name] = [parameter value]
```
 The pound symbol (#) is used for comments, and any text after it is ignored by the file parser.

 The following table lists all of the input parameters, separated into the same groupings in the input file above:

 ### Dimensionality
| Name          | Options | Required | Default | Description |
| --------------|---------|----------|---------|-------------|
| Number of dimensions | 2, 3 | yes | n/a | The number of dimensions for the simulation. |

### Computational Domain
| Name          | Options | Required | Default | Description |
| --------------|---------|----------|---------|----------------------------------------------------|
| Domain size X | Any positive real number | yes | n/a | The size of the domain in the x direction. |
| Domain size Y | Any positive real number | yes | n/a | The size of the domain in the y direction. |
| Domain size Z | Any positive real number | yes | n/a | The size of the domain in the z direction. |

### Element Parameters
| Name          | Options | Required | Default | Description |
| --------------|---------|----------|---------|----------------------------------------------------|
|Subdivisions X | Any positive integer | no | 1 | The number of mesh subdivisions in the x direction to control the element aspect ratio. The mesh size is \f$2^{(Refine Factor)} \times Subdivisions\f$ in each direction.
Subdivisions Y | Any positive integer | no | 1 | The number of mesh subdivisions in the y direction to control the element aspect ratio.The mesh size is \f$2^{(Refine Factor)} \times Subdivisions\f$ in each direction.
Subdivisions Z | Any positive integer | no | 1 | The number of mesh subdivisions in the z direction to control the element aspect ratio. The mesh size is \f$2^{(Refine Factor)} \times Subdivisions\f$ in each direction.
Refine factor | Any non-negative integer | yes | n/a | The number of initial refinements of the mesh. The mesh size is \f$2^{(Refine Factor)} \times Subdivisions\f$ in each direction. While in principle the mesh could be entirely controlled by the number of subdivisons, computational performance is best when the majority of the refinement is done via the Refine factor.
Element degree | 1, 2, 3 | no | 1 | The polynomial order of the elements. The spatial order of accuracy is one plus the degree.

### Mesh Adaptivity (Optional)
| Name          | Options | Required | Default | Description |
| --------------|---------|----------|---------|----------------------------------------------------|
Mesh adaptivity | Boolean | no | false | Controls whether mesh adaptivity is enabled.
Max refinement level | Any non-negative integer | no | -1 | The maximum number of local refinements during adaptive meshing. This parameter does not need to be specified if mesh adaptivity is disabled, but the default value will cause an error if mesh adaptivity is enabled.
Min refinement level | Any non-negative integer | no | -1 | The minimum number of local refinements during adaptive meshing. This parameter does not need to be specified if mesh adaptivity is disabled, but the default value will cause an error if mesh adaptivity is enabled.
Refinement criteria fields | Comma separated list of variable names | no | 0 | The indices of the variables that will determine the mesh refinement. The variable names are determined by the names given in equations.h.
Refinement window max | Comma separated list of real numbers | no |  | The mesh refines where the specified variables are between an upper and lower bound. This specifies the upper bound.
Refinement window min | Comma separated list of real numbers | no |  | The mesh refines where the specified variables are between an upper and lower bound. This specifies the lower bound.
Steps between remeshing operations | Positive integer | no | 1 | The number of time steps between mesh refinement operations.

### Time Stepping
| Name          | Options | Required | Default | Description |
| --------------|---------|----------|---------|----------------------------------------------------|
Time step | Any positive real number | yes | n/a | The time step size for the simulation.
Number of time steps | Any non-negative integer | no | -1 | The number of time steps until the simulation stops. Either this or the simulation end time must be specified. If both are specified, the simulation will end when the first condition is reached.
Simulation end time | Any non-negative real number | no | -0.1 | The simulated time when until the simulation stops. Either this or the number of time steps must be specified. If both are specified, the simulation will end when the first condition is reached.

### Linear Solver Parameters (Optional)
| Name          | Options | Required | Default | Description |
| --------------|---------|----------|---------|----------------------------------------------------|
Use absolute convergence tolerance | Boolean | no | false | Sets whether to use an absolute tolerance for the linear solver (versus a relative tolerance).
Solver tolerance value | Any positive real number | no | 1e-3 | The tolerance for the linear solver. If an absolute tolerance is used, the iterative solver stops when the L2 norm of the residual drops below the tolerance. If a relative tolerance is used, the iterative solver stops when the L2 norm of the residual has dropped by a factor equal to the tolerance.
Maximum allowed solver iterations | Any positive integer | no | 10000 | The maximum number of iterations for the linear solver, if this number of iterations is reached, the solver stops regardless of the tolerance value.

### Output (Optional)
| Name          | Options | Required | Default | Description |
| --------------|---------|----------|---------|----------------------------------------------------|
Output condition | EQUAL_SPACING, LOG_SPACING, N_PER_DECADE, LIST | no | EQUAL_SPACING | This sets the spacing between the times the simulation outputs the model variables. EQUAL_SPACING spaces them equally, LOG_SPACING spaces them  \f$10^{n/(outputs) \log(time steps)}\f$, N_PER_DECADE allows the user to set how many times the simulation outputs per power of ten iterations, and LIST outputs at a user-given list of time step numbers.
Number of outputs | Any non-negative integer | no | 10 | The number of inputs if the output condition is EQUAL_SPACING. The number of outputs if the output condition is N_PER_DECADE. Ignored for the other output conditions.
List of time steps to output | Comma-separated list of non-negative integers | no | 0 | The list of time steps to output, used for the LIST output condition and ignored for the others.
Output file name (base) | String | no | solution | The name for the output file, before the time step and processor info are added.
Output file type | vtu, vtk | no | vtu | The output file type (currently limited to either vtu or vtk).
Output separate files per process | Boolean | no | false | Whether to output separate vtu files for each process in a parallel calculation (automatically set to true for vtk files). Separate files may decrease the time spent outputting results but may increase file tranfer times, as well as cluttering directories.
Skip print steps | Any positive integer | no | 1 | The number of time steps between updates to the terminal window (1 is every time step).

### Checkpoints (Optional)
| Name          | Options | Required | Default | Description |
| --------------|---------|----------|---------|----------------------------------------------------|
Load from a checkpoint | Boolean | no | false | Whether to start the simulation from the checkpoint of another simulation. See Note 2 below for the details of the checkpoint/restart system.
Checkpoint condition | EQUAL_SPACING, LOG_SPACING, N_PER_DECADE, LIST | no | EQUAL_SPACING | This sets the spacing between the times the simulation outputs the model variables. EQUAL_SPACING spaces them equally, LOG_SPACING spaces them  \f$10^{n/(outputs) \log(time steps)}\f$, N_PER_DECADE allows the user to set how many times the simulation outputs per power of ten iterations, and LIST outputs at a user-given list of time step numbers.
Number of checkpoints | Any non-negative integer | no | 1 | The number of inputs if the output condition is EQUAL_SPACING. The number of outputs if the output condition is N_PER_DECADE. Ignored for the other output conditions.
List of time steps to save checkpoints | Comma-separated list of non-negative integers | no | 0 | The list of time steps to create checkpoints, used for the LIST output condition and ignored for the others.

### Boundary Conditions
| Name          | Options | Required | Default | Description |
| --------------|---------|----------|---------|----------------------------------------------------|
Boundary condition for variable [variable name] | NATURAL, DIRICHLET, NON_UNIFORM_DIRICHLET, PERIODIC | yes | n/a | Sets the boundary condition for each scalar variable, using the variable name from equations.h. One line is required for each scalar  variable. See Note 1 below for details.
Boundary condition for variable [variable name], component [direction] | NATURAL, DIRICHLET, NON_UNIFORM_DIRICHLET, PERIODIC | yes | n/a | Sets the boundary condition for each vector variable, using the variable name from equations.h. One line is required for each vector variable. See Note 1 below for details.

### Loading Initial Conditions from File (Optional)
| Name          | Options | Required | Default | Description |
| --------------|---------|----------|---------|----------------------------------------------------|
Load initial conditions | Comma-separated list of booleans | no | false | One true/false flag for each variable for whether the initial condition should be loaded from a vtk file. Currently, only scalar fields can be read in. See Note 3 below for details.
Load parallel file | Comma-separated list of booleans | no | false | One true/false flag for each variable for whether each processor should read the initial conditions from a seperate file. See Note 3 below for details.  
File names | Comma-separated list of strings | no |  | The name of the vtk file to be read for each variable, ignored for variables where the initial conditions isn't read in from a file. Often, all of the variables will read from the same vtk file. See Note 3 below for details.
Variable names in the files | Comma-separated list of strings | no |  | What each variable is named in the file being loaded. See Note 3 below for details.

### Shared Nucleation Parameters (Optional)
| Name          | Options | Required | Default | Description |
| --------------|---------|----------|---------|----------------------------------------------------|
Time steps between nucleation attempts | Any positive integer | no | 100  | The number of time steps between nucleation attempts. This parameter is shared among all nucleating variables. See Note 4 below for details.
Minimum allowed distance between nuclei | Any non-negative real number | no | 2\f$\times\f$ the largest nucleus semiaxis  | The minimum allowed distance between nuclei centers during a single nucleation attempt. This parameter is shared among all nucleating variables. See Note 4 below for details.
Order parameter cutoff value | Any non-negative real number | no | 0.01  | The minimum allowed value of the sum of all nucleating variable fields where nucleation is allowed to occur. Implemented to prevent nucleation inside existing particles. This parameter is shared among all nucleating variables. See Note 4 below for details.
Time steps between nucleation attempts | Any positive integer | no | 100  | The number of time steps between nucleation attempts. This parameter is shared among all nucleating variables. See Note 4 below for details.

### Nucleation Parameters for Each Nucleating Variable (Optional)
| Name          | Options | Required | Default | Description |
| --------------|---------|----------|---------|----------------------------------------------------|
Nucleus semiaxes (x, y ,z) | Three positive real numbers | no |  | The semiaxes of the ellipsoidal nuclei to be seeded. See Note 4 below for details.
Nucleus rotation in degrees (x, y, z) | Three real numbers | no | | The rotation of the nuclei placed with the explicit nucleation algorithm. The rotations are given with respect to the normal direction using intrinsic Tait-Bryan angles. Positive rotations correspond to a  counter-clockwise rotation when looking along the positive axis of interest. All three angles must be specified regardless of problem dimension.
Freeze zone semiaxes (x, y ,z) | Three positive real numbers | no |  | The semiaxes for the ellipsoidal region where the nucleus is frozen after seeding. See Note 4 below for details.
Freeze time following nucleation | Any non-negative real number | no | 0 | The amount of time the nucleus is frozen after seeding. See Note 4 below for details.
Nucleation-free border thickness | Any non-negative real number | no | 0 | The size of the buffer region where nucleation is not allowed unless the boundary conditions are periodic. See Note 4 below for details.

### Model constants (Optional)
| Name          | Options | Required | Default | Description |
| --------------|---------|----------|---------|----------------------------------------------------|
Model constant [constant name] | value followed by a comma then a type | no |  | Sets the value of a constant defined for that particular application. The allowed types are DOUBLE, INT, BOOL, TENSOR, and [symmetry] ELASTIC CONSTANTS where [symmetry] is ISOTROPIC, TRANSVERSE, ORTHOTROPIC, or ANISOTROPIC. See Note 5 below for details.

## Note 1: Boundary Conditions
The boundary condition must be set for each variable, where each variable is given by its name, as defined in equations.h. The four boundary condition types are NATURAL, DIRICHLET, NON_UNIFORM_DIRICHLET and PERIODIC. If all of the boundaries have the same boundary condition, only one boundary condition type needs to be given. If multiple boundary condition types are needed, give a comma-separated list of the types. The order is the miniumum of x, maximum of x, minimum of y, maximum of y, minimum of z, maximum of z (i.e left, right, bottom, top in 2D and left, right, bottom, top, front, back in 3D). The value of a Dirichlet BC is specfied in the following way -- DIRCHILET: val -- where 'val' is the desired value. If the boundary condition is NON_UNIFORM_DIRICHLET, the boundary condition should be specified in the appropriate function in 'ICs_and_BCs.cc'. For vector variables, one boundary condition should be specified for each component.

Example 1: All periodic BCs for variable 'c'
```
set Boundary condition for variable c = PERIODIC
```

Example 2: Zero-derivative BCs on the left and right, Dirichlet BCs with value 1.5 on the top and bottom for variable 'n' in 2D
```
set Boundary condition for variable n = NATURAL, NATURAL, DIRICHLET: 1.5,
 DIRICHLET: 1.5
```

Example 3: All periodic BCs for the y component of the vector variable 'u'
```
set Boundary condition for variable u, component y = PERIODIC
```

## Note 2: Checkpoint/Restart
The checkpoint/restart simulation allows one to continue from a previous simulation by loading the mesh and variable values from that previous simulation. One use of this system is to continue a simulation that stopped part of the way through, due to a hardware failure, running out of the allotted time on a cluster, etc. A second use is to use one simulation as the initial condition for another. Note that the simulated time carries over from the checkpoint. Thus if one ran a simulation to completion but wanted to see further evolution, one could load from the checkpoint created at the end of that simulation, but the desired number of time steps or simulation end time would have to be increased. Currently, the checkpoint system always read from files named ''restart.mesh'', ''restart.mesh.info'', and ''restart.time.info''. When a checkpoint is created, the previous checkpoint files have ''.old'' appended to their file names. To load from these older checkpoint files, the newer ones should be deleted (or moved) and the ''.old'' should be deleted from the names of the older files. An example of using the checkpoint/restart system can be seen in the 'dendriticSolidification' application.

### Note 3: Loading Initial Conditions from File
Currently, initial conditions can only be read from vtk files (_not_ vtu files). Some variables can have initial conditions read from file and others can be specified in ICs_and_BCs.cc in the same application. For each variable, the user must set whether the file(s) to be read are in serial format, where all processors read from the same file, or parallel format, where all processors read from different files. If parallel format is selected, the desired domain decomposition must between the files and the simulation to be run. For non-adaptive meshes this will be true if the same number of cores is the same between the simulation that generated the vtk files and the simulation reading the vtk files. For adaptive meshes, obtaining the same domain decomposition may be difficult and merging parallel vtk files into a single file is likely the best approach. This process is planned to be cleaner in future versions of PRISMS-PF. An example of loading initial conditions from file can be seen in the 'allenCahn' application, using the 'parameters_pfield.in' input file.

### Note 4: Nucleation
PRISMS-PF includes the capability for explicitly placing nuclei over the course of a simulation using an approach similar to the one described in the following publication:

Jokisaari, Permann, and Thornton, A nucleation algorithm for the coupled conserved-nonconserved phase field model, \emph{Computational Materials Science}, 112, (2016).

A nucleus of an non-conserved order parameter is placed within the computational domain determined by a probability given by the 'getNucleationProbability' function in the 'nucleation.cc' file. The variables that are allowed to nucleate are specified in the 'loadVariableAttributes' function in the 'equations.cc' file, as are the variable values needed to calculate the nucleation probability. The determination of when and where a nucleus should be seeded is determined in the core PRISMS-PF library, but the actual placement of the nucleus by modifying one of the model fields is expected to be performed in the 'explicitEquationRHS' function in the 'equations.cc' file. The placement of the nucleus is aided by the 'weightedDistanceFromNucleusCenter' function in the core library. The mobility for the nucleated field should be set to zero in the region surrounding the nucleus (the ''freeze zone'') for a specified amount of time (the ''freeze time''). Examples of nucleating particles can be found in the 'nucleationModel' and 'nucleationModel_preferential' applications.

Currently, the nucleation parameters are one of the few places where the subsection' command is used in the input file.  This command is used to set nucleation parameters separately for each nucleating variable. Each subsection is opened by ''subsection Nucleation parameters:'' followed by the variable name as set in 'equations.cc'. The end of each subsection block is concluded with the 'end' command. For example, if there are two nucleating order parameters **n1** and **n2**, the nucleation parameters section could look as follows:
```
# =================================================================================
# Set the nucleation parameters
# =================================================================================

set Time steps between nucleation attempts = 30
set Minimum allowed distance between nuclei = 50.0
set Order parameter cutoff value = 0.01

subsection Nucleation parameters: n1
    set Nucleus semiaxes (x, y, z) = 10, 5, 5
    set Nucleus rotation in degrees (x, y, z) = 0, 0, 60
    set Freeze zone semiaxes (x, y, z) = 15, 7.5, 7.5
    set Freeze time following nucleation = 20
    set Nucleation-free border thickness = 15
end

subsection Nucleation parameters: n2
    set Nucleus semiaxes (x, y, z) = 20, 10, 10
    set Nucleus rotation in degrees (x, y, z) = 0, 0, 0
    set Freeze zone semiaxes (x, y, z) = 30, 15, 15
    set Freeze time following nucleation = 40
    set Nucleation-free border thickness = 30
end
```
The first three lines set the shared nucleation parameters. Next comes the subsection block for the variable **n1** and the subsection block for the variable **n2**.

## Note 5: Model constants
Each application specifies its own set of model constants. These are most often used in the residual equations in the 'equations.cc' file, although they may also be used to specify initial conditions, non-uniform Dirichlet boundary conditions, nucleation probabilties, etc. Currently, five types of model constants are accepted: DOUBLE, INT, BOOL, TENSOR, and ELASTIC CONSTANTS. The use of these different types is as follows:

- DOUBLE: For individual real numbers
- INT: For individual integers
- BOOL: For individual booleans (true/false)
- TENSOR: For either rank 1 tensors (i.e. vectors) or rank 2 tensors (i.e. matrices) with a size of the number of dimensions. These are assumed to be real numbers. Each row should be given by a comma-separated list in parentheses. For example, a 3D vector would be given as ```(2.5, 1.2, 0.1)``` and a 2D matrix would be given as: ```((4.1, 2.0),(1.6, 5.2))```.
- {[symmetry]} ELASTIC CONSTANTS: For sets of elastic constants, given as a vector. The symmetry options are ISOTROPIC, TRANSVERSE, ORTHOTROPIC, or ANISOTROPIC. The number of entries for the different symmetries are given below. PRISMS-PF converts these sets of elastic constants into a stiffness matrix.

The number and order of the entries for the elastic constants are (where \f$C_{ij}\f$ are entries in the stiffness matrix):

- ISOTROPIC (2D/3D): 2 constants (Young's Modulus, Poisson's Ratio)
- TRANSVERSE (3D): 5 constants (\f$C_{11}\f$, \f$C_{33}\f$, \f$C_{44}\f$, \f$C_{12}\f$, \f$C_{13}\f$)
- ORTHOTROPIC (3D): 9 constants (\f$C_{11}\f$, \f$C_{22}\f$, \f$C_{33}\f$, \f$C_{44}\f$, \f$C_{55}\f$, \f$C_{66}\f$, \f$C_{12}\f$, \f$C_{13}\f$, \f$C_{23}\f$)
- ANISOTROPIC (2D/3D): 21 constants (\f$C_{11}\f$, \f$C_{22}\f$, \f$C_{33}\f$, \f$C_{44}\f$, \f$C_{55}\f$, \f$C_{66}\f$, \f$C_{12}\f$, \f$C_{13}\f$, \f$C_{14}\f$, \f$C_{15}\f$, \f$C_{16}\f$, \f$C_{23}\f$, \f$C_{24}\f$, \f$C_{25}\f$, \f$C_{26}\f$, \f$C_{34}\f$, \f$C_{35}\f$, \f$C_{36}\f$, \f$C_{45}\f$, \f$C_{46}\f$, \f$C_{56}\f$)

Each model constant in `parameters.in' must have a counterpart in the appropriate section of the `customPDE.h' file.

# Other App Files {#app_files}

This section details the files that define each application. The norm is that substantial modifications to these files consistute a new application (in contrast to simply making changes to the input file ('parameters.in'). Modifying these files may require some knowledge of C++.

A very brief description of the purpose of each of these files is below, an in-depth discussion can be found in the following subsections:
- equations.cc: Specifies the attributes of the model variables and the model's residual equations
- ICs\_and\_BCs.cc: Specifies the initial conditions for each variable (unless the initial condition is read from file) and non-uniform Dirichlet boundary conditions, if applicable.
- postprocess.cc: Specifies variable other than the primary model variables to output, as well as the expressions to derive these variables.
- nucleation.cc: Contains the function that determines the probability density of nucleation.
- customPDE.h: Contains the prototypes of the functions for the application, also contains declarations of the model constants given in the input file.
- main.cc: Main C++ function that controls the flow of the simulation. Identical for all the example applications and it is unlikely that users will need to modify it.

In all of these files, the user can access user inputs from parameters.in via the ''userInputs'' object (e.g. the domain size, the time step size). See the documentation entry for userInputParameters for a list of variable names inside ''userInputs''.

## equations.cc
The file ''equations.cc'' contains a list of the variables in the model equations and their attributes as well as the residuals for the model equations. The file contains four functions: loadVariableAttributes, explicitEquationRHS, nonExplicitEquationRHS, and equationLHS.

To modify the functions in this file, one needs to be familiar with the weak form of the governing equations. In PRISMS-PF, the governing equations are expressed in two terms. The first is the part of the integrand that is multiplied by the test function (marked by 'eq' with the subscript of the variable in the example below). The second is the part of the integrand that multiplied by the gradient of the test function (marked by 'eqx' with the subscript of the variable in the example below). For the coupled Cahn-Hilliard/Allen-Cahn system, the governing equations are

\f$\int_{\Omega}   w  \eta^{n+1}  ~dV =\int_{\Omega}  w  \left( \underbrace{\eta^{n} - \Delta t M_{\eta}~ ((f_{\beta,c}^n-f_{\alpha,c}^n)H_{,\eta}^n)}_{eq_{\eta}} \right)+ \nabla w \cdot \underbrace{(- \Delta t M_{\eta}\kappa) \nabla \eta^{n}}_{eqx_{\eta}} ~dV \f$

and

\f$\int_{\Omega}   w  c^{n+1}  ~dV = \int_{\Omega}   w \underbrace{c^{n}}_{eq_c} +  \nabla w   \underbrace{(-\Delta t M_{c})~ [~(f_{\alpha,cc}^n(1-H^{n+1})+f_{\beta,cc}^n H^{n+1}) \nabla c + ~((f_{\beta,c}^n-f_{\alpha,c}^n)H^{n+1}_{,\eta} \nabla \eta) ] }_{eqx_{c}} ~dV \f$

for the Allen-Cahn and Cahn-Hilliard equation, respectively. Each of the terms in the governing equation is marked with an underbrace. The terms multiplied by the test function are referred to as the value terms and the terms multiplied by the gradient of the test function are referred to as the gradient terms.

### loadVariableAttributes
Here is the 'loadVariableAttributes' function for the coupled Allen-Cahn/Cahn-Hilliard example application:
```
void variableAttributeLoader::loadVariableAttributes(){
	// Variable 0
	set_variable_name				(0,"c");
	set_variable_type				(0,SCALAR);
	set_variable_equation_type		(0,EXPLICIT_TIME_DEPENDENT);

    set_dependencies_value_term_RHS(0, "c");
    set_dependencies_gradient_term_RHS(0, "n,grad(c)");

    // Variable 1
	set_variable_name				(1,"n");
	set_variable_type				(1,SCALAR);
	set_variable_equation_type		(1,EXPLICIT_TIME_DEPENDENT);

    set_dependencies_value_term_RHS(1, "c,n");
    set_dependencies_gradient_term_RHS(1, "grad(n)");
}
```

This function specifies the model variables and their attributes. In this case, the two model variables are the concentration, **c**, and the order parameter, **n**. Here, **c** is listed as the zeroth variable and **n** is listed as the first. For each variable, a series of attributes are set using a series of C++ function calls. The following table lists the functions and a description of the attributes they set:

| Function          | Options | Required | Default | Description |
| --------------|---------|----------|---------|----------------------------------------------------|
set_variable_name | String | no | var  | Sets the name of the variable. This name is used in 'parameters.in' as well as during output.
set_variable_type | SCALAR, VECTOR | no | SCALAR  | Sets whether the variable is a scalar or a vector.
set_variable_equation_type | EXPLICIT_TIME_DEPENDENT, AUXILIARY, TIME_INDEPENDENT | no | EXPLICIT_TIME_DEPENDENT  | Sets whether the governing equation for the variable is a time-dependent PDE (EXPLICIT_TIME_DEPENDENT), a time-independent PDE that does not require a linear solve (AUXILIARY) or a time independent PDE that does require a (non)linear solve (TIME_INDEPENDENT).
set_dependencies_value_term_RHS | [a string] | yes | | Sets which variables and their derivatives are needed to calculate the value term for the RHS. Variables are referenced by their names. First derivatives are referenced by ```grad``` and then the variable name in parentheses. Second derivatives are referenced by ```hess``` and then the variable name in parentheses.
set_dependencies_gradient_term_RHS | [a string] | yes | | Sets which variables and their derivatives are needed to calculate the gradient term for the RHS. Variables are referenced by their names. First derivatives are referenced by ```grad``` and then the variable name in parentheses. Second derivatives are referenced by ```hess``` and then the variable name in parentheses.
set_dependencies_value_term_LHS | [a string] | no | "" | Sets which variables and their derivatives are needed to calculate the value term for the RHS. Variables are referenced by their names. First derivatives are referenced by ```grad``` and then the variable name in parentheses. Second derivatives are referenced by ```hess``` and then the variable name in parentheses. (Only needed for TIME_INDEPENDENT equations.)
set_dependencies_gradient_term_LHS | [a string] | no | "" | Sets which variables and their derivatives are needed to calculate the gradient term for the RHS. Variables are referenced by their names. First derivatives are referenced by ```grad``` and then the variable name in parentheses. Second derivatives are referenced by ```hess``` and then the variable name in parentheses. (Only needed for TIME_INDEPENDENT equations.)
set_allowed_to_nucleate | Boolean | no | false | Sets whether the nucleation algorithms should be activated for this variable. (Only needed when nucleation is desired).
set_need_value_nucleation | Boolean | no | false | Sets whether the value of the variable is needed to calculate the nucleation probability in the 'nucleation.cc' file. (Only needed when nucleation is desired).

Some of these function calls are not present in the 'equations.cc' file for the coupledCahnHilliardAllenCahn application. Use of the LHS function calls can be found in the preciptiateEvolution application (among others) and use the nucleation function calls can be found the nucleationModel and nucleationModel_preferential applications.

### explicitEquationRHS
The 'explicitEquationRHS' function is where the terms in the RHS of the governing equations for EXPLICIT_TIME_DEPENDENT equations are entered. The terms in the RHS of other equations are entered into the 'nonExplicitEquationRHS' function. Here is the 'explicitEquationRHS' function from the coupled Allen-Cahn/Cahn-Hilliard example application:
```
template <int dim, int degree>
void customPDE<dim,degree>::explicitEquationRHS(variableContainer<dim,degree,dealii::VectorizedArray<double> > & variable_list,
				 dealii::Point<dim, dealii::VectorizedArray<double> > q_point_loc) const {

// --- Getting the values and derivatives of the model variables ---

//c
scalarvalueType c = variable_list.get_scalar_value(0);
scalargradType cx = variable_list.get_scalar_gradient(0);

//n
scalarvalueType n = variable_list.get_scalar_value(1);
scalargradType nx = variable_list.get_scalar_gradient(1);

// --- Setting the expressions for the terms in the governing equations ---

// Free energy for each phase and their first and second derivatives
scalarvalueType fa = (-1.6704-4.776*c+5.1622*c*c-2.7375*c*c*c+1.3687*c*c*c*c);
scalarvalueType fac = (-4.776 + 10.3244*c - 8.2125*c*c + 5.4748*c*c*c);
scalarvalueType facc = (10.3244-16.425*c+16.4244*c*c);
scalarvalueType fb = (5.0*c*c-5.9746*c-1.5924);
scalarvalueType fbc = (10.0*c-5.9746);
scalarvalueType fbcc = constV(10.0);

// Interpolation function and its derivative
scalarvalueType h = (10.0*n*n*n-15.0*n*n*n*n+6.0*n*n*n*n*n);
scalarvalueType hn = (30.0*n*n-60.0*n*n*n+30.0*n*n*n*n);

// Residual equations
scalargradType mux = ( cx*((1.0-h)*facc+h*fbcc) + nx*((fbc-fac)*hn) );
scalarvalueType eq_c = c;
scalargradType eqx_c = (constV(-Mc*userInputs.dtValue)*mux);
scalarvalueType eq_n = (n-constV(userInputs.dtValue*Mn)*(fb-fa)*hn);
scalargradType eqx_n = (constV(-userInputs.dtValue*Kn*Mn)*nx);


// --- Submitting the terms for the governing equations ---

// Terms for the equation to evolve the concentration
variable_list.set_scalar_value_term_RHS(0,eq_c);
variable_list.set_scalar_gradient_term_RHS(0,eqx_c);

// Terms for the equation to evolve the order parameter
variable_list.set_scalar_value_term_RHS(1,eq_n);
variable_list.set_scalar_gradient_term_RHS(1,eqx_n);

}
```

In this function the residuals at a particular quadrature point are calculated. The inputs to this function are a list of the model variable values and derivatives, variable_list and a point giving access to (x,y,z) coordinates, q_point_loc. The residual terms are added to variable_list as the output. The first few lines of the function set more convenient names for the variables and their derivatives. By convention, the value of the variable is denoted by the variable name (c for the concentration and n for the structural order parameter in this case), the list of first derivatives is denoted by the variable name followed by an ''x'', and second derivatives are denoted by the variable name followed by ''xx''. Each variable in variable can be accessed by the index it was given in loadVariableAttributes. The variable value and the derivatives can be accessed through the get_scalar_value,  get_scalar_gradient, and get_scalar_hessian object members for scalar variables and the  get_vector_value,  get_vector_gradient, and get_vector_hessian functions. The data type for the value of a scalar variable is scalarvalueType (a scalar), the data type for the first derivatives of a scalar variable is scalargradType (a vector with a length equal to the number of dimensions), and the data type for the second derivatives of a scalar variable is scalarhessType (a matrix with a size equal to the number of dimensions by the number of dimensions). For vector variables, the data types are vectorvalueType (a vector with length equal to the number of dimensions), vectorgradType (a matrix with a size equal to the number of dimensions by the number of dimensions), and vectorgradType (a rank-three tensor with a size in each direction equal to the number of dimensions).

After the nicknames for the field variables are set, the residual terms are calculated (including some intermediate variables, such as the free energies and the interpolation functions in the example above). These use the same six data types discussed in the preceding paragraph. The model variables given in 'parameters.in' can be used to define the residual terms.

Finally, the terms for the governing equations are submitted to variable_list, using the functions set_scalar_value_term_RHS, set_scalar_gradient_term_RHS, set_vector_value_term_RHS, and set_vector_gradient_term_RHS, once again referring the variables by their index.

The nickname step can be skipped, if desired, although the code is generally more readable with nicknames (and the performance overhead is minimal). An example without nicknames for the fields can be found in the grainGrowth application, where loops over the ten variables are easier to construct when not declaring all of the variables at once. One word of caution, though: calling set_scalar_value_term_RHS and set_scalar_gradient_term_RHS overwrites the value and gradient of the variable, respectively (and same for the variants for vectors). Thus, either the residuals need to be cached and then set after all the residuals have been calculated (as is done in the grainGrowth application) or the values and gradients need to be cached before setting the residuals (as is done with the standard nicknaming approach).

#### A note on types
The deal.II library uses a data structure called a VectorizedArray to store the variable values and their derivatives. This data structured in optimized for modern vectorized processors, giving a substantial speedup in some cases. However, this data structure can complicate things slightly. One complicating factor is that VectorizedArrays can't always be added, subtracted, multiplied, or divided with more standard data types like doubles. For this reason, you will see the ''constV(argument)'' function scattered throughout the code. This function turns a non-VectorizedArray into a VectorizedArray. To be safe, you can always encase non-VectorizedArrays with ''constV()'' when they share an operation with a VectorizedArray. A second complication is that not all of the standard mathematical operations are available for VectorizedArrays. The basic trigonometric functions are available, as are exponentials and square roots. However, hyperbolic tangents are not. If needed, they must be constructed from exponents (or by iterating through the VectorizedArray, see below). A third complication is that conditional statements involving VectorizedArrays are not allowed. To perform a conditional statement, you must iterate through the VectorizedArray. To do still, construct a for loop where the maximum index is [variable name].n_array_elements. For examples of this, refer to the postprocessing file for the grainGrowth app or the seedNucleus function in 'equations.cc' in the nucleationModel app. For more details on the deal.II implementation of VectorizedArrays (including a list of mathematical operations that are allowed), please visit [the relevant deal.II documentation page](https://www.dealii.org/8.4.0/doxygen/deal.II/classVectorizedArray.html).


### nonExplicitEquationRHS
The 'nonExplicitEquationRHS' function is where the terms in the RHS of the governing equations for AUXILIARY and TIME_INDEPENDENT equations are entered. The terms in the RHS of EXPLICIT_TIME_DEPENDENT equations are entered into the 'explicitEquationRHS' function. The structure and use is otherwise identical to 'explicitEquationRHS'. Examples of apps where this function is used include cahnHilliard (for an AUXILIARY equation) and MgNd_precipitate_single_Bppp (for AUXILIARY and TIME_INDEPENDENT equations).

### equationLHS
In the coupledCahnHilliardAllenCahn app, the equationLHS function is empty because it is only needed for TIME_INDEPENDENT PDEs (or more specifically, when a non-trivial matrix inversion needs to be performed). Here we go through the equationLHS function from the precipitateEvolution application, where the equation for mechanical equilibrium is TIME_INDEPENDENT.

From the formulation file in the precipitateEvolution application, the governing equation for the mechanical displacement is:
\f$
R(u) = \int_{\Omega}   \nabla w :  C(\eta_1, \eta_2, \eta_3) : \left( \epsilon - \epsilon^0(c,\eta_1, \eta_2, \eta_3)\right) ~dV = 0
\f$

In PRISMS-PF, matrix inversion problems are always written as Newton's method iterations. For linear equations, like the one above, the solution is reached in a single Newton step. The reason for this approach is two-fold. First, it provides an identical user interface for linear and nonlinear problems. Second, it enables the efficient handling of constraints for when inhomogeneous Dirichlet boundary conditions are used.

To write the above equations in terms of a Newton iteration, the solution, \f$u\f$, can be written as the sum of an initial guess, \f$u_0\f$, and an update, \f$\Delta u\f$:

\f$
R(u) = R(u_0 + \Delta u) = R(u_0) +  \int_{\Omega} \left. \frac{\delta R(u)}{\delta u}\right|_{u=u_0} \Delta u ~dV = 0
\f$

In this case, the equation is linear and the variation derivative is trivial:

\f$
R(u_0 + \Delta u) =  \int_{\Omega}   \nabla w :  C(\eta_1, \eta_2, \eta_3) : \left( \epsilon(u_0 + \Delta u) - \epsilon^0(c,\eta_1, \eta_2, \eta_3)\right) ~dV = \int_{\Omega}   \nabla w :  C(\eta_1, \eta_2, \eta_3) : \left( \epsilon(u_0) + \epsilon(\Delta u) - \epsilon^0(c,\eta_1, \eta_2, \eta_3)\right) ~dV = 0
\f$

Rearranging yields:

\f$
\int_{\Omega} \nabla w : \underbrace{C : \nabla (\epsilon(\Delta u))}_{eqx_{u}^{LHS}} dV = -\int_{\Omega}   \nabla w : \underbrace{C :(\epsilon(u_0)-\epsilon^0)}_{eqx_{u}^{RHS}} ~dV
\f$

The above values of \f$eqx_{u}^{LHS}\f$ and \f$eqx_{u}^{RHS}\f$ are used to define the residuals in the equations.h file. A similar process can be undertaken for other TIME_INDEPENDENT problems.

Here is the 'equationLHS' function from the precipitateEvolution example application:
```
template <int dim, int degree>
void customPDE<dim,degree>::equationLHS(variableContainer<dim,degree,dealii::VectorizedArray<double> > & variable_list,
		dealii::Point<dim, dealii::VectorizedArray<double> > q_point_loc) const {

// --- Getting the values and derivatives of the model variables ---

//n1
scalarvalueType n1 = variable_list.get_scalar_value(1);

//n2
scalarvalueType n2 = variable_list.get_scalar_value(2);

//n3
scalarvalueType n3 = variable_list.get_scalar_value(3);

//u
vectorgradType Dux = variable_list.get_change_in_vector_gradient(4);


// --- Setting the expressions for the terms in the governing equations ---

vectorgradType eqx_Du;

// Interpolation functions

scalarvalueType h1V = (10.0*n1*n1*n1-15.0*n1*n1*n1*n1+6.0*n1*n1*n1*n1*n1);
scalarvalueType h2V = (10.0*n2*n2*n2-15.0*n2*n2*n2*n2+6.0*n2*n2*n2*n2*n2);
scalarvalueType h3V = (10.0*n3*n3*n3-15.0*n3*n3*n3*n3+6.0*n3*n3*n3*n3*n3);

// Take advantage of E being simply 0.5*(ux + transpose(ux)) and use the dealii "symmetrize" function
dealii::Tensor<2, dim, dealii::VectorizedArray<double> > E;
E = symmetrize(Dux);

// Compute stress tensor (which is equal to the residual, Rux)
if (n_dependent_stiffness == true){
	dealii::Tensor<2, CIJ_tensor_size, dealii::VectorizedArray<double> > CIJ_combined;
	CIJ_combined = CIJ_Mg*(constV(1.0)-h1V-h2V-h3V) + CIJ_Beta*(h1V+h2V+h3V);

	computeStress<dim>(CIJ_combined, E, eqx_Du);
}
else{
	computeStress<dim>(CIJ_Mg, E, eqx_Du);
}

// --- Submitting the terms for the governing equations ---

variable_list.set_vector_gradient_term_LHS(4,eqx_Du);

}
```

Like the functions for the RHS, equationLHS takes variables_list and q_point_loc as inputs. However, because only one TIME_INDEPENDENT equation is solved at a time, the output are the terms for a single governing equation. As in \emph{residualRHS}, the model variable values and derivatives are given convenient names at the start of the file. The equationLHS function has a new option for getting variable values and derivatives, here one can access the change in the value of one of the variables or its derivatives. The function name is ```get_change_in_vector_gradient``` above. The other options are ```get_change_in_scalar_value```, ```get_change_in_scalar_gradient```, ```get_change_in_scalar_hessian```, ```get_change_in_vector_value```, and ```get_change_in_vector_hessian```.  Similar to the RHS functions, the value of eqx_Du is set and then used to set the appropriate residual for variables_list (in this case the vector residual term for the fourth variable).

If multiple TIME_INDEPENDENT equations need to be solved, you will need to use conditional statements to calculate the proper residual depending on the elliptic equation being solved. The index of the field being solved can be accessed using the ```this-$>$currentFieldIndex``` statement.

## ICs_and_BCs.cc
The file ''ICs_and_BCs.cc'' contains the initial conditions for the primary variables as well as the expressions for non-uniform Dirichlet boundary conditions, if present. The file contains two functions: setInitialCondition and setNonUniformDirichletBCs.

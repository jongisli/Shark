LinAlg: Eigenvalues, Inverses and Systems of Linear Equations
=============================================================
LinAlg offers a basis set of linear algebra routines especially for
solving systems of linear equations, matrix inversion, and eigenvalue problems.

A lot of the routines presented below use the :doc:`ATLAS <vector_matrix>` backend when available.
It is highly recommended to use this backend. 

Solving Systems of Linear Equations
-------------------------------------------------------------
A system of linear equations has the forms

.. math::
  A\vec{x}=\vec{b} \\
  AX=B
  
and we want to find the solution for :math:`x` or :math:`X`. The routines to choose depend highly on the form of :math:`A`. To use the solvers the file 
``shark/LinAlg/solveSystem.h`` must be included.

The easiest form for an triangular system arises when :math:`A` is a triangular matrix. For a lower triangular matrix all entries above the diagonal 
are zero. Accordingly, for an upper triangular matrix the elements below the diagonal are zero. Solving this kind of problems is that easy 
that the system can be solved without additional memory by using :math:`\vec{b}` or :math:`B` to store the results. To solve an equation 
of this form, use doxy:`solveTriangularSystemInPlace`::

  bool isLowerTriangular = true;
  solveTriangularSystemInPlace(isLowerTriangular,A,b);
  
The boolean parameter describes whether :math:`A` is lower triangular or not. Choosing the parameter correct is crucial.
Be aware that a transpose of a lower triangular :math:`A` is upper triangular::

  solveTriangularSystemInPlace(false,trans(A),b);

The same rules apply for the matrix versions::

  solveTriangularSystemInPlace(true,A,B);
  solveTriangularSystemInPlace(false,trans(A),B);
  
This routine is very important as a building block for more complex systems of linear equations. If :math:`A` is a symmetric matrix, it can be decomposed using
Cholesky decomposition into

.. math::
  A = CC^T

where :math:`C` is a lower triangular matrix. With this decomposition, the system can be solved with the routines above::

  solveTriangularSystemInPlace(true,C,B);
  solveTriangularSystemInPlace(false,trans(C),B);

Of course, there is a more direct way to solve this system, namely by using :doxy:`solveSymmSystem`::

  solveSymmSystem(A,x,b);
  solveSymmSystem(A,X,B);
  
As you can see, this is not an in-place method anymore. The solution is numerically stable and quite exact.
Be aware, that :math:`A` must be positive definite. If :math:`A` does not have full rank, the solution to this
system will be garbage. However, for most problems, this is not an issue.

If your matrix is not symmetric, but still has full rank, the more general :doxy:`solveSystem` can be used::

  solveSystem(A,x,b);
  solveSystem(A,X,B);
  
This function uses a LU-decomposition to solve the system. This is not as numerically stable as the symmetric version and is more expensive to compute.
There is no solver when :math:`A` does not have full rank.

Inverse of a Matrix
-----------------------------------------------------------

Routines for computing inverses are provided in the file
 ``shark/LinAlg/LinAlg.h``.

For Symmetric positive definite matrices, the inverse can be calculated using
:doxy:`invertSymmPositiveDefinite`::

  invertSymmPositiveDefinite(invA,A);
  
For general matrices with full rank, the more general :doxy:`invert` is available which also offers a convenient syntax::

  inverseA=inverse(A);


.. note:: 
  Often computing inverses is  not required, but only the solution to a system of
  linear equations. In this case, solving the system directly is faster
  and numerically more stable. 




Eigenvalues of a Matrix
--------------------------------------------------------------------
Eigenvalue equations are special linear equations of the form:

.. math::
  Ax=\lambda x

The lambdas are called eigenvalues and the :math:`x` are the eigenvectors with :math:`||x||=1`. If A is symmetric, :doxy:`eigensymm` can be used to solve this system::

  eigensymm(A,X,lambda);
  
X stores all eigenvectors of A as columns and lambda is a vector of the corresponding eigenvalues. The *i*-th column 
of X corresponds to the *i*-th element of lambda. Eigenvalues are sorted in descending order.

Singular Value Decomposition
--------------------------------------------------------------------
The Singular Value Decomposition decomposes :math:`A` into

.. math::
  A=UWV^T
  
Such that the columns of :math:`U` and :math:`V` are orthonormal and
:math:`W` is diagonal. 
The matrix :math:`A` does not need to have full rank. In fact, it does not even need to be quadratic. The
SVD can be used by including ``shark/LinAlg/svd.h``: ::

  svd(A,U,V,W);

The SVD is often used to compute pseudo-inverses, matrices which are only left or right inverses to :math:`A`.

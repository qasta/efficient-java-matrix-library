# Introduction #

Solving a system of linear equations is the fundamental problem in linear algebra.  A linear system is any system than can be expressed in this format:

_A_**_x_ = _b_**

where _A_ is m by n, _x_ is n by o, and _b_ is m by o.  Most of the time o=1.  There are numerous ways to solve such system.  Which technique is optimal depends on the size and structure of the matrix.

In EJML the easiest way to solve a linear system is by calling CommonOps.solve() when using DenseMatrix64F or SimpleMatrix.solve().  LinearSolvers can be used if more knowledge is available about the type of matrix, more control is needed over memory, or if the solutions quality needs to be checked.  It is also possible to directly call decomposition algorithms, but this technique is not recommended since it requires much more knowledge about the algorithms and the specific implementation used.

Linear solvers will in general fail (with some notable exceptions) to produce a meaning full solution if the 'A' matrix is singular.  When 'A' is singular then there is an infinite number of solutions. This failure might be detected or it might not.  In fact EJML does a better job of reporting failures than all other libraries tested in [java matrix benchmark](http://code.google.com/p/java-matrix-benchmark/).  Depending on which interface is used to solve the system, there are different ways to check for this degenerate case.  These are discussed below.

# Solving with CommonOps and SimpleMatrix #

The recommended method for quick development and for novice users is calling CommonOps.solve() or if using SimpleMatrix calling SimpleMatrix.solve().  Below are examples of how to solve a linear system:

Using DenseMatrix64F:
```
DenseMatrix64F A = new DenseMatrix64F(m,n);
DenseMatrix64F x = new DenseMatrix64F(n,1);
DenseMatrix64F b = new DenseMatrix64F(m,1);

.... code ....

if( !CommonOps.solve(A,b,x) ) {
  throw new IllegalArgument("Singular matrix");
}
```

Using SimpleMatrix:
```
SimpleMatrix A = new SimpleMatrix(m,n);
SimpleMatrix b = new SimpleMatrix(m,1);

.... code ...

try {
  SimpleMatrix x = A.solve(b);
} catch ( SingularMatrixException e ) {
  throw new IllegalArgument("Singular matrix");
}

```

**NOTE** while there is error checking above for singular matrices, these will not catch "nearly" singular systems.  If it is not known before hand if 'A' is singular or not then using a LinearSolver instead should be considered.  An alternative approach that is commonly used is to compute the SVD of the system and to check the magnitude of the smallest singular value.  Computing the SVD is expensive.

# Solving with LinearSolver #

While it is easier to use CommonOps.solve(), that function has limited capabilities and can be inefficient.  If more control is needed or the system might be singular then LinearSolvers should be used. The LinearSolver interface is designed to be easy to use and to provide most of the power that comes from using decompositions directly would provide.  Below is a summary of the interface:
```
public interface LinearSolver< T extends Matrix64F> {

    public boolean setA( T A );

    public T getA();

    public double quality();

    public void solve( T B , T X );

    public void invert( T A_inv );

    public boolean modifiesA();

    public boolean modifiesB();
}
```

There are several different implementations of LinearSolver for different types of matrices.  A new LinearSolver should be created by invoking static functions inside of LinearSolverFactory.  LinearSolverFactory will select the best algorithm to use and is updated as newer algorithms are added.

Two steps are required to solve a system with a LinearSolver, as is shown below:
```
LinearSolver<DenseMatrix64F> solver = LinearSolverFactory.leastSquares();

if( !solver.setA(A) ) {
  throw new IllegalArgument("Singular matrix");
}

if( solver.quality() <= 1e-8 )
  throw new IllegalArgument("Nearly singular matrix");

solver.solve(b,x);
```


Additional functions included in LinearSolver are:

  * invert()
    * Will invert a matrix more efficiently than solve() can.
  * quality()
    * Returns a positive number which if it is small indicates a singular or nearly singular system system.  Much faster to compute than the SVD.
  * modifiesA() and modifiesB()
    * To reduce memory requirements, most LinearSolvers will modify the 'A' and store the decomposition inside of it.  Some do the same for 'B'  These function tell the user if the inputs are being modified or not.

If the input matrices 'A' and 'B' should not be modified then LinearSolverSafe is a convenient way to ensure that precondition:
```
   LinearSolver<DenseMatrix64F> solver = LinearSolverFactory.leastSquares();
   solver = new LinearSolverSafe<DenseMatrix64F>(solver);
```


## AdjustableLinearSolver ##

In situations where rows from the linear system are added or removed (see [PolynomialFitExample](PolynomialFitExample.md)) an AdjustableLinearSolver can be used to efficiently resolve the modified system.  AdjustableLinearSolver is an extension of LinearSolver that adds addRowToA() and removeRowFromA().  These functions add and remove rows from A respectively.  After being involved the solution can be recomputed by calling solve() again.

```
AdjustableLinearSolver solver = LinearSolverFactory.adjustable();

if( !solver.setA(A) ) {
  throw new IllegalArgument("Singular matrix");
}

solver.solve(b,x);

// add a row
double row[] = new double[N];

... code ...

solver.addRowToA(row,2);

.... adjust b and x ....

solver.solve(b,x); 

// remove a row
solver.removeRowFromA(7);

.... adjust b and x ....


solver.solve(b,x);
```
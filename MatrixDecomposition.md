# Introduction #

In linear algebra a matrix decomposition is often used as an intermediate step when solving a problem.  They allow a more complex problem to be reformulated into a more simplistic problem or provide insight into the structure of a matrix.  EJML provides the following matrix decompositions:

| **Decomposition** | **DenseMatrix64F** | **BlockMatrix64F** |
|:------------------|:-------------------|:-------------------|
| LU | Yes |  |
| Cholesky L`*`L<sup>T</sup> and R<sup>T</sup>`*`R | Yes | Yes |
| Cholesky L`*`D`*`L<sup>T</sup> | Yes |  |
| QR | Yes | Yes |
| QR Column Pivot | Yes |  |
| Singular Value Decomposition (SVD) | Yes |  |
| Generalized Eigen Value | Yes |  |
| Symmetric Eigen Value | Yes | Yes |
| Bidiagonal | Yes |  |
| Tridiagonal | Yes | Yes |
| Hessenberg | Yes |  |

Columns for DenseMatrix64F and BlockMatrix64F specify if a native decomposition exists for that data structure.  Specialized helper functions are provided for SVD and EVD.

# Solving Using Matrix Decompositions #

Decompositions such as LU and QR are most commonly used to solve a linear system.  A common mistake in EJML is to directly decompose the matrix instead of using a LinearSolver.  LinearSolvers significantly simplify the process of solving a linear system and will automatically be updated as new algorithms are added.

For more information on LinearSolvers see the wikipage at [SolvingLinearSystems](SolvingLinearSystems.md).

# SimpleMatrix #

SimpleMatrix has easy to an use interface built in for SVD and EVD:
```
SimpleSVD svd = A.svd();
SimpleEVD evd = A.eig();

SimpleMatrix U = svd.getU();
```
where A is a SimpleMatrix.

As with most operators in SimpleMatrix, it is possible to chain a decompositions with other commands.  For instance, to print the singular values in a matrix:
```
A.svd().getW().extractDiag().transpose().print();
```

Other decompositions can be performed by using accessing the internal DenseMatrix64F and using the decompositions shown in the following section below.  The following is an example of applying a Cholesky decomposition.
```
CholeskyDecomposition<DenseMatrix64F> chol = DecompositionFactory.chol(A.numRows(),true);
    
if( !chol.decompose(A.getMatrix()))
   throw new RuntimeException("Cholesky failed!");
        
SimpleMatrix L = SimpleMatrix.wrap(chol.getT(null));
```

# DecompositionFactory #

The best way to create a matrix decomposition is by using DecompositionFactory.  Directly instantiating a decomposition is discouraged because of the added complexity.  DecompositionFactory is updated as new and faster algorithms are added.

```
public interface DecompositionInterface<T extends Matrix64F> {
    /**
     * Computes the decomposition of the input matrix.  Depending on the implementation
     * the input matrix might be stored internally or modified.  If it is modified then
     * the function {@link #inputModified()} will return true and the matrix should not be
     * modified until the decomposition is no longer needed.
     *
     * @param orig The matrix which is being decomposed.  Modification is implementation dependent.
     * @return Returns if it was able to decompose the matrix.
     */
    public boolean decompose( T orig );

    /**
     * Is the input matrix to {@link #decompose(org.ejml.data.DenseMatrix64F)} is modified during
     * the decomposition process.
     *
     * @return true if the input matrix to decompose() is modified.
     */
    public boolean inputModified();
}
```

Most decompositions in EJML implement DecompositionInterface.  To decompose the matrix simply call decompose(A), where A is the matrix being decomposed.  It returns true if there was no error while decomposing and false otherwise.  While in general you can trust the results if true is returned some algorithms can have faults that are not reported.  This is true for all linear algebra libraries.

To minimize memory usage, most decompositions will modify the original matrix passed into decompose().  Call inputModified() to determine if the input matrix is modified or not.  If it is modified, and you do not wish it to be modified, just pass in a copy of the original instead.

Below is an example of how to compute the SVD of a matrix:
```
    void decompositionExample( DenseMatrix64F A ) {
        SingularValueDecomposition<DenseMatrix64F> svd = DecompositionFactory.svd(A.numRows,A.numCols);

        if( !svd.decompose(A) )
            throw new RuntimeException("Decomposition failed");

        DenseMatrix64F U = svd.getU(null,false);
        DenseMatrix64F W = svd.getW(null);
        DenseMatrix64F V = svd.getV(null,false);

    }
```
Note how it checks the returned value from decompose.

In addition DecompositionFactory provides functions for computing the quality of a decomposition.  Being able measure the decomposition's quality is an important way to validate its correctness.  It works by "reconstructing" the original matrix then computing the difference between the reconstruction and the original.  Smaller the quality is the better the decomposition is. With an ideal value of being around 1e-15 in most cases.

```
if( DecompositionFactory.quality(A,svd) > 1e-3 )
    throw new RuntimeException("Bad decomposition");
```

List of functions in DecompositionFactory

| **Decomposition** | **Code** |
|:------------------|:---------|
| LU | DecompositionFactory.lu() |
| QR | DecompositionFactory.qr() |
| QRP | DecompositionFactory.qrp() |
| Cholesky | DecompositionFactory.chol() |
| Cholesky LDL | DecompositionFactory.cholLDL() |
| SVD | DecompositionFactory.svd() |
| Eigen | DecompositionFactory.eig() |


# Helper Functions for SVD and EVD #

Two classes SingularOps and EigenOps have been provided for extracting useful information from these decompositions or to provide highly specialized ways of computing the decompositions.  Below is a list of more common uses of these functions:

SingularOps
  * descendingOrder()
    * In EJML the ordering of the returned singular values is not in general guaranteed.  This function will reorder the U,W,V matrices such that the singular values are in the standard largest to smallest ordering.
  * nullSpace()
    * Computes the null space from the provided decomposition.
  * rank()
    * Returns the matrix's rank.
  * nullity()
    * Returns the matrix's nullity.

EigenOps
  * computeEigenValue()
    * Given an eigen vector compute its eigenvalue.
  * computeEigenVector()
    * Given an eigenvalue compute its eigenvector.
  * boundLargestEigenValue()
    * Returns a lower and upper bound for the largest eigenvalue.
  * createMatrixD() and createMatrixV()
    * Reformats the results such that two matrices (D and V) contain the eigenvalues and eigenvectors respectively.  This is similar to the format used by other libraries such as Jama.
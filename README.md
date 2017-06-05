# Matrix Multiplication

At the base of article is performance research of matrix multiplication.
Let`s take a few input data:

- Multiplication matrix: [A]*[B] = [C], where [A], [B], [C] - square matrix
- Size of each matrix is 2048 x 2048
- Matrix is dense. So, zero's values is very small 
- Programming language: [Go](https://golang.org/)
- Laptop precossor for experiments: [Intel(R) Core(TM) i5-3230M CPU @2.6 GHz](https://ark.intel.com/ru/products/72164/Intel-Core-i5-3230M-Processor-3M-Cache-up-to-3_20-GHz-rPGA)
> The number of CPU cores : 2
>
> The number of threads: 4
>
> CPU cache : 3MB

You can see the theory in [Wikipedia](https://en.wikipedia.org/wiki/Matrix_multiplication).
Shortly:
Matrix is table of values, for example:

```
[ 2 3 ]
[ 4 5 ]
```

Now, you see the square matrix with 2 rows and 2 columns. Like you understood, values can be any.
So, a little example for matrix multiplication looks like that:

```
[ 2 3 ]  *  [ 1 6 ]  =  [ 2*1+3*9 2*6+3*8 ] =  [ 29 36 ]
[ 4 5 ]     [ 9 8 ]     [ 4*1+5*9 4*6+5*8 ]    [ 49 64 ]
```

Yes, we will optimize the classic algorithm of multiplication with O(n^3).

We can start to analyze present design of algorithm in exist libraries.

Code in project [JAMA : A Java Matrix Package](http://math.nist.gov/javanumerics/jama/)
```java
public class Matrix implements ... {
   ...
   public Matrix times (Matrix B) {
	  ...
      Matrix X = new Matrix(m,B.n);
      double[][] C = X.getArray();
      double[] Bcolj = new double[n];       // Buffer of column matrix B
      for (int j = 0; j < B.n; j++) {
         for (int k = 0; k < n; k++) {
            Bcolj[k] = B.A[k][j];           // Put in buffer values
         }
         for (int i = 0; i < m; i++) {
            double[] Arowi = A[i];
            double s = 0;
            for (int k = 0; k < n; k++) {
               s += Arowi[k]*Bcolj[k];     // Multiplication
            }
            C[i][j] = s;
         }
      }
      return X;
   }
   ...
}
```
Comment - at the future, I will show the reason - Why buffer of column for matrix B is good? But we create the better.

Code in project [go.matrix](https://github.com/skelterjohn/go.matrix/blob/daa59528eefd43623a4c8e36373a86f9eef870a2/arithmetic.go)
```golang
func ParallelProduct(A, B MatrixRO) (C *DenseMatrix) {
	...
	C = Zeros(A.Rows(), B.Cols())

	in := make(chan int)
	quit := make(chan bool)

	dotRowCol := func() {
		for {
			select {
			case i := <-in:
				sums := make([]float64, B.Cols())      // Array for intermediante results
				for k := 0; k < A.Cols(); k++ {
					for j := 0; j < B.Cols(); j++ {
						sums[j] += A.Get(i, k) * B.Get(k, j)
					}
				}
				for j := 0; j < B.Cols(); j++ {
					C.Set(i, j, sums[j])
				}
			case <-quit:
				return
			}
		}
	}

	threads := 2                                      // Is it looks strange, because I can more then 2 processors

	for i := 0; i < threads; i++ {
		go dotRowCol()
	}

	for i := 0; i < A.Rows(); i++ {
		in <- i
	}

	for i := 0; i < threads; i++ {
		quit <- true
	}

	return
}
```
Comments:
- Strange, but it is true, amount of threads put in code. The best way to use actual processors on user computer
- We don`t see buffer
- We see using array for intermadiante results. Now, it is not clear - it is good or not. We will see.

# Stop theory, more practic, more benchmarks



TODO

add more tests
add test for one single matrix
add tests for deep matrix


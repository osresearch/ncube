# 7 USING THE SYSTEM
## 7.1 Introduction
In order to program the system effectively a user must think of a particular problem as a set of smaller problems. In some applications, particularly in the physical sciences, this is relatively easy. Most science problems involve solving equations in a 2 or 3 dimensional space and one can simply divide the space into pieces and solve the equations in these divided spaces, matching the solutions at the edges. In other applications dividing the problem into smaller pieces may not be as straight forward. However, almost all large problems must be subdivided, just to be manageable. A large proportion of important problems can be solved effectively on the system of the present invention.

One difference between the system of the present invention and the more traditional "pipelined" approach to high performance computing is that one must divide both the program and the data into smaller pieces. This is sometimes more difficult than having many programs working on a large shared memory, but it more accurately models the real physical world of local phenomena and it is the only way to overcome the memory speed bottleneck of shared memory systems.

Many problems will require a modification of the hypercube interconnection scheme. Therefore, the following section describes how to map the hypercube onto some of these different interconnection patterns.

## 7.2 Hypercube Mappings
The hypercube interconnection system was chosen for three main reasons:

1) It is a recursive scheme, so it will usually be easy to write programs that are independent of the order of the hypercube. This facilitates time and space sharing by the operating system.

2) It maps directly onto the most important common interconnection patterns (i.e. 1,2,3,4 dimensional lattices, "perfect shuffle" and trees).

3) It is so extensively interconnected that it gives a good approximation to maximal (every processor connected to all others) interconnection. Thus, if a problem does not have an obvious interconnection structure, it will normally be acceptable to assign subproblems arbitrarily to hypercube nodes and let the communication software take care of routing messages.

Since many physical problems split naturally onto lattices, algorithms for mapping the hypercube onto grids up to dimension 4 will be described.

### 7.2.1 Gray Code
All of the hypercube mappings are most easily described using some variant of Gray code. A gray code is a one-to-one mapping between integers such that the binary representations of the images of any two consecutive integers differ in exactly one place. The domain is assumed to be finite and the largest and smallest integers in the domain are "consecutive". One example of a gray code for three bit integers is ##STR49##

One may intuitively see that a gray code is important by realizing that in a hypercube, if processor x is connected to processor y, then the binary representations of x and y must differ in exactly one place. There is a unique gray code implemented with the following algorithm:

1) let x be a nonnegative number represented in binary

2) let y be x after a right shift by one place

3) then z=x XOR y is the gray code image of x

The code to implement this algorithm is (X must be nonnegative):

MOVM X,R0

SFTW #-1,R0

XORW X,R0

MOVW R0,Z

As will be seen below, the inverse mapping is also needed In other words if z is a gray code image we will need to be able to calculate the value x whose gray code is z. The inverse mapping for the gray code algorithm given above is implemented in the following code: ##STR50##

Although the gray code above is unique, there are many mappings between integers that have the property of mapping consecutive integers to images that differ in one place. Any such mapping can be used in the algorithms described below.

### 7.2.2 One Dimensional Grid
A one dimensional grid (or ring) is simply a string of interconnected processors as shown ##STR51##

This interconnection is often useful when the problem does not have to solved in real time and can be broken down into steps that can be pipelined. Non-realtime filtering is an example. The mapping in this case is simply any gray code as described in section 7.2.1. Thus, if F is the gray code and G is its inverse then the neighbors of processor x are:

F(G(x)-1) and F(G(x)+1).
7.2.3 Two Dimensional Grid
Steady state problems involving two space dimensions (e.g. boundary value problems) map naturally onto a two dimensional grid. To define a two dimensional mapping, assume that the grid is 2**M in the x direction by 2**N in the y direction, then the processor number at location (x,y) is

2**M*F(y)+F(x).
Also, if a processor number is k=2**M*z+w then its neighbors are:

2**M*F(G(z))+F(G(w)-1)=2**M*z+F(G(w)-1)
2**M*F(G(z))+F(G(w)+1)=2**M*z+F(G(w)+1)
2**M*F(G(z)-1)+F(G(w))=2**M*F(G(z)-1)+w
2**M*F(G(z)+1)+F(G(w))=2**M*F(G(z)+1)+w
By using a slightly more complicated scheme where neighbors are determined by shuffling the bits of the images one has a mapping where the neighbors of k are fixed independent of the size of the hypercube.

### 7.2.4 Three Dimensional Grid
Many real physical problems are mappe onto three dimensional grids. An example is fluid flow whether in airplane wing design (turbulent, compressible flow) or oil reservoir modeling (incompressible flow). A three dimensional mapping is analogous to the two dimensional case except the processor ID numbers are divided into three parts instead of two and there are six neighbors instead of four.

### 7.2.5 Four Dimensional Grid
If a problem involves both time and space it may be conveniently mapped onto a four dimensional grid. In this case a processor ID number is divided into four parts and each processor has eight neighbors.

## 7.3 Computational Example (User Programming)
In this section programming to solve a typical problem is presented.

### 7.3.1 Simultaneous Linear Equations
Simultaneous linear equation problems are categorized according to the structure of the matrix representing the problem. The two main types are:

1) dense--where the matrix is mostly full of nonzero elements.

2) sparse--where the matrix is mostly zero. Within these categories matrices are further subdivided into:

1) general--where the matrix elements have no perceivable structure.

2) symmetric, positive definite--where the matrix elements are symmetric across the main diagonal and the determinants of all the principal minors are positive.

Below programming is shown for solving the dense matrix Gaussian elimination. The method involves computing factors of a matrix so that the problem is solvable in two steps. For example, suppose we want to solve

Ax=b
where A is the matrix of coefficients, x is the unknown and b is the known vector. In the factorization methods we compute

A=CD
where C and D have some special structure (orthogonal or triangular). Then the equations are solved by computing y and then x as shown

Cy=b
Dx=y
The structure of C and D make the systems above easy to solve.

#### 7.3.1.1 Hypercube Mapping
In the algorithms for dense matrices the matrix is broken up into equal
rectangles (as close to squares as possible). Also the hypercube is
mapped onto a two dimensional grid. Thus, in the ideal case where there
are M^2 processors and the matrix is N by N, then we would put subblocks
of size N/M by N/M in each processor. The process is illustrated below
where the subscripts refer to both the subblock of the matrix and to
the processor. ##STR52##

#### 7.3.1.3 Gaussian Elimination
Gaussian elimination with partial pivoting is a relatively stable and fast method to solve a set of dense linear equations that have no special structure. This method computes a factorization of A called LU (i.e. A=LU where L is lower triangular and U is upper triangular). Gaussian elimination can be used efficiently on the system of the present invention and pivoting does not slow the algorithm down appreciably.

The following user program computes L and U using partial pivoting with actual row interchanges. The elements of L and U replace the elements of A.

Given:
 * M = Hypercube order
 * PN = Processor Number
 * N = Size of Aij subblock (N by N)
 * A = subblock of coefficient matrix

Calculate:
 * I     = Row coordinate of processor and matrix subblock
 * J     = Column coordinate of processor and matrix subblock
 * NN     = North Neighbor (-1 if no neighbor)
 * EN     = East Neighbor (-1 if no neighbor)
 * SN     = South Neighbor (-1 if no neighbor)
 * WN     = West Neighbor (-1 if no neighbor)

Allocate:
 * RBMAX(N)   = Row buffer for maximum row
 * RBTEM(N)   = Row buffer for interchanges

Program:
```
FOR X = 1 TO MIN(I,J)			;the kth row and column of processors
					;do k stages of elimination
	FOR Y = 1 to N			;N rows must be used for elimination
					;at each stage

		L = 1			;from here to where noted below is for
					;pivot selection and row interchange

		IF (I = J) THEN L = Y	;if the processor is on the
					;diagonal and the last stage
					;is reached, the loops must
					;start at Y

		IF (X = J) THEN		;we are in the pivot column and
					;K is calculated as the index of
					;the row with the maximum pivot
			T = 0
			K = 1
			FOR W = L TO N
				IF (ABS(A(W,Y) > T) THEN
					K = W
					T = ABS(A(W,Y))
		ELSE
			RECEIVE(WN,K,1)	;K is received by processors
					;to the right of the pivot
					;column

		SEND(EN,K,1)		;K is sent to the right after either
					;being computed or received

		IF (I = M) THEN		;we are in the last row and must
					;start the row selection by
					;setting the buffer to the Kth
					;row
			FOR W = 1 TO N
				RBMAX(W) = A(K,W)
		ELSE			;if we are a row above the last then we
					;receive the maximum row from the processor
					;below us, compare it with our maximum row,
					;perform the exchange if necessary and send
					;the current maximum row up to the next row
					;of processors
			RECEIVE(SN,RBMAX,N)
			IF (X = J) THEN	;EXCH is true if a row
					;interchange is necessary
				EXCH = (ABS(RBMAX(Y) > ABS(A(K,Y))
			ELSE RECEIVE(WN,EXCH,1)
			SEND(EN,EXCH,1)	;if we are going to inter-
			SEND(SN,EXCH,1)	;change both the processors
					;to the right and below
					;must know
			IF (EXCH) THEN
				FOR W = 1 TO N
					RBTEM(W) = A(K,W)
					A(K,W) = RBMAX(W)
				SEND(SN,RBTEM,N)
			ELSE
				FOR W = 1 TO N
					RBMAX(W) = A(K,W)

		IF (X >< I) THEN	;if we are not in the top row
					;we must continue the process
					;of selection by interchanging
					;and sending the maximum row to
					;the processors above
			SEND(NN,RBMAX,N)
			RECEIVE(NN,REPL,1)
			IF (REPL) THEN
				RECEIVE(NN,RBTEM,N)
				FOR W = 1 TO N
					A(K,W) = RBTEM(W)

		IF (X = I) THEN
			SEND(SN,RBMAX,N) ;if we are in
					;the top row we
					;only send a
					;row but
		ELSE			;otherwise we both send and receive a row
			RECEIVE(NN,RBMAX,N)
			SEND(SN,RBMAX,N) ;this completes the section
					;of the program that
					;selects and interchanges
					;the pivot row

		FOR Z = L to N		;the rest of the program performs
					;the decomposition using the row
					;selected above
			IF (X = J) THEN
				PIVOT = - (A(Z,Y)/RBMAX(Y))
				A(Z,Y) = PIVOT
			ELSE RECEIVE(WN,PIVOT,1)
			SEND(EN,PIVOT,1)
			IF (X = J) THEN L = L + 1
			FOR W = L TO N
				A(Z,W) = A(Z,W) + RBMAX(W) * PIVOT
```


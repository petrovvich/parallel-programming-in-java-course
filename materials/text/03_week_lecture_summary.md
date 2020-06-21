## 3.1 Lecture Summary
### 3 Loop Parallelism
#### 3.1 Parallel Loops

Lecture Summary: In this lecture, we learned different ways of expressing parallel loops. The most general way is to 
think of each iteration of a parallel loop as an async task, with a finish construct encompassing all iterations. 
This approach can support general cases such as parallelization of the following pointer-chasing while loop 
(in pseudocode):

```
    finish {
        for (p = head; p != null ; p = p.next) async compute(p);
     }
```

However, further efficiencies can be gained by paying attention to counted-for loops for which the number of iterations 
is known on entry to the loop (before the loop executes its first iteration). We then learned the forall notation for 
expressing parallel counted-for loops, such as in the following vector addition statement (in pseudocode):

```
    forall (i : [0:n-1]) a[i] = b[i] + c[i]
```

We also discussed the fact that Java streams can be an elegant way of specifying parallel loop computations that 
produce a single output array, e.g., by rewriting the vector addition statement as follows:

```
    a = IntStream.rangeClosed(0, N-1).parallel().toArray(i -> b[i] + c[i]);
```

In summary, streams are a convenient notation for parallel loops with at most one output array, but the forall notation 
is more convenient for loops that create/update multiple output arrays, as is the case in many scientific computations. 
For generality, we will use the forall notation for parallel loops in the remainder of this module.

Optional Reading:

1. [Tutorial on Executing Streams in Parallel](https://docs.oracle.com/javase/tutorial/collections/streams/parallelism.html#executing_streams_in_parallel)


## 3.2 Lecture Summary
### 3 Loop Parallelism
#### 3.2 Parallel Matrix Multiplication

In this lecture, we reminded ourselves of the formula for multiplying two n × n matrices, a and b, to obtain a product 
matrix, c, of size n × n:

```
c [ i ][ j ] = ∑k=0n−1\sum_{k=0}^{n-1}∑k=0n−1​ a[ i ][ k ] ∗ b[ k ][ j ]
```

This formula can be easily translated to a simple sequential algorithm for matrix multiplication as follows 
(with pseudocode for counted-for loops):

```
for(i : [0:n-1]) {
  for(j : [0:n-1]) { c[i][j] = 0;
    for(k : [0:n-1]) {
      c[i][j] = c[i][j] + a[i][k]*b[k][j]
    }
  }
}
```

he interesting question now is: which of the for-i, for-j and for-k loops can be converted to forall loops, i.e., can 
be executed in parallel? Upon a close inspection, we can see that it is safe to convert for-i and for-j into forall 
loops, but for-k must remain a sequential loop to avoid data races. There are some trickier ways to also exploit 
parallelism in the for-k loop, but they rely on the observation that summation is algebraically associative even though 
it is computationally non-associative.


Optional Reading:

1. [Wikipedia article on Matrix multiplication algorithm](https://en.wikipedia.org/wiki/Matrix_multiplication_algorithm)


## 3.3 Lecture Summary
### 3 Loop Parallelism
#### 3.3 Barriers in Parallel Loops

Lecture Summary: In this lecture, we learned the barrier construct through a simple example that began with the following 
forall parallel loop (in pseudocode):

```
forall (i : [0:n-1]) {
        myId = lookup(i); // convert int to a string 
        print HELLO, myId;
        print BYE, myId;
}
```

We discussed the fact that the HELLO’s and BYE’s from different forall iterations may be interleaved in the printed 
output, e.g., some HELLO’s may follow some BYE’s. Then, we showed how inserting a barrier between the two print 
statements could ensure that all HELLO’s would be printed before any BYE’s.

Thus, barriers extend a parallel loop by dividing its execution into a sequence of phases. While it may be possible to 
write a separate forall loop for each phase, it is both more convenient and more efficient to instead insert barriers in a 
single forall loop, e.g., we would need to create an intermediate data structure to communicate the myId values from one 
forall to another forall if we split the above forall into two (using the notation next) loops. Barriers are a fundamental 
construct for parallel loops that are used in a majority of real-world parallel applications.
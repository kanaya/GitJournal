# Slice and General Slice for Scheme

When we handle digital images on a computer, we'd better use a
combination of one-dimensional array and slice, but not
multi-dimensional array, for better efficiency and readability of your
code.

The slice is a kind of knife that can cut out submatrix of the
original matrix which actually is a single-dimensional array. To
illustrate, imagine there is a matrix like this.

> a00 a01 a02 a03
> a11 a11 a12 a13
> a21 a21 a22 a23

The memory of the computer forms one-dimensional array. Thus the above
matrix (two-dimensional array) must be stored as one-dimensional
array. Were you a C programmer, you'd store the matrix as follows.

> a00 a01 a02 a03 a10 a11 a12 a13 a20 a21 a22 a23

Imagine we want to take (slice) out submatrix (subarray) from this
array. It is easy to slice out a00 a01 a02 a03 by taking the first
four elements of the array, however, slicing out a00 a10 a20 seems a
little bit difficult.

> a00 a01 a02 a03
> a10 a11 a12 a13
> a20 a21 a22 a23

Remember that this matrix is stored as following one-dimensional
array.

> a00 a01 a02 a03 a10 a11 a12 a13 a20 a21 a22 a23

(This memory allocation is used by pseudo two-dimensional matrix of C
language, and FORTRAN, C++ STL uses other allocation like a00 a10 a20
a01 a11 ...) The sequence of a00 a10 a20 is a "slice of offset 0, size
3, stride 4" of the array, or "3 elements interleaved by 4 from 0th
element" in natural language.

FORTRAN, which has long historical background, and C++, which is the
most popular language among programmers, have special supports to
slicing of one-dimensional array. Here is an example.

    #include <valarray>
    using namespace std;
    void foo()
    {
      valarray<double> a(100); // use valarray, but not vector, for slicing arrays
      a[slice(0, 20, 5)] = 1.0;
      ...
    }

In this example we are assigning 1.0 to 20 elements interleaved by 5.

For referencial transparency, which is the most important aspect of
functional programming, let us have a function that creates a new
sliced array.

    template <class T> valarray<T> *new_sliced_array(size_t start, size_t size, size_t stride, const valarray<T> &a)
    {
      return new valarray<T>(a[slice(start, size, stride)]);
    }

A general slice is an extended version of regular slice. It extracts
m-dimensional subarray from n-dimensional original array. For example,
extracting from the following original array

> a00 a01 a02 a03
> a10 a11 a12 a13
> a20 a21 a22 a23

and obtaining the following array

> a00 a01
> a10 a11

is what the general slice does.

> a00 a01 a02 a03 a10 a11 a12 a13 a20 a21 a22 a23

The following example shows how to use the general slice in C++.

    template <class T> valarray<T> *new_gsliced_array(size_t start, valarray<size_t> sizes, valarray<size_t> strides, const valarray<T> &a)
    {
      return new valarray<T>(a[gslice(start, sizes, strides)]);
    }

The slices are indispensable in scientific research including image
processing, however, we, computer scientists, are not very relaxed
when we only have FORTRAN and C++ as practical tools. This is why we
create slice procedure in Scheme.

    (define (slice start size stride array)
      (if (<= size 0)
        '() ; nothing is left
        (cons
		  (ref array start) ; the first element
		  (slice (+ start stride) (- size 1) stride array)))) ; rest elements

The second-order slice can be done as follows.

    (define (slice2 start size0 size1 stride0 stride1 array)
      (if (<= size1 0)
        '() ; nothing is left
        (append ; we use append instead of cons
          (slice start size0 stride0 array) ; the first slice
          (slice2 (+ start stride1) size0 (- size1 1) stride0 stirde1 array)))) ; rest slices

We can now create a general slice from above examples, but let us have a third-order slice just in case.

    (define (slice3 start size0 size1 size2 stride0 stride1 stride2 array)
      (if (<= size2 0)
        '() ; nothing is left
        (append
		  (slice2 start size0 size1 stride0 stride1 array) ; the first 2nd-order slice
		  (slice3 (+ start stride2) size0 size1 (- size2 1) stride0 stride1 stride2 array)))) ; rest 2nd-order slices

Here is a general slice.

    (define (gslice start sizes strides array)
      (if (<= (car sizes) 0)
        '() ; nothing is left
        (if (or (null? (cdr strides)) (null? (cdr sizes)))
          (slice start (car sizes) (car strides) array) ; the first slice
          (append ; rest slices
            (gslice start (cdr sizes) (cdr strides) array) ; the first general slice
            (gslice ; rest general slices
              (+ start (car strides))
              (cons (- (car sizes) 1) (cdr sizes))
              strides
              array)))))

We can use this procedure as follows.

    (print (gslice start '(size3 size2 size1 size0) '(stride3 stride2 stride1 stride0) array))

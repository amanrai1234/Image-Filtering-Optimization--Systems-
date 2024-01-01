# -Image-filtering-optimization

This file contains the instructions to run the program and the changes made to it to improve the parallel performance.

Instruction to run the .c program in the csug (or any linux) system:

Step-1: Compile the program:
gcc image_blur.c -o image_blur -pthread
(Note: The -pthread need to be added in the end since we are using pthreads in this program.

[In Case the above line fails to compile the code, then run the below command]
gcc image_blur.c -o image_blur -lpthread

Step-2: Run the program:
To run the program just execute the following command (optional -m, -n, -h, -v paramters can be added):
./image_blur


Optimizations done on the program:

- The intensity calculation and filter smooting calculations were already well parallelized in the program, which the initialization and creation of the image feature was not parallelized, although it could be parallelised.
- The optimisation that I did was, I moved the for loop which assigned random values to each pixel of the image (serially) to a function called ImageInitialize(), and I parallelised that function using pthreads to the anount of threads specified by the user (32 by default). This showed a lot of improvement in the execution time.
- It is to be noted that, I initially tried to parallelze the entire for loop, including the malloc function for each of the rows as well. But this did not show any improvement in the code, but instead increased the excecution time of the code with the increasing number if threads since, malloc() is not re-entrant, and that causing deadlock and there by instead slowing down the execution. Hence I retained the malloc() functions alone in the main single thread for loop, which runs exectly before our newly created ImageInitialize() function is called.
- Also I faced a very critical issue with these changes. The code will run on my laptop, but it will created deadlock in the CSUG systems. The main reason for this is that, the compatibility of rand() function in C, with pthread is very poor, and it does not properly parallelize in all platforms. Hence I created a self-defined rand() function which has lesser time complexity and can safely run in parallel. This random function was created using the concept of Linear-feedback shift register (reference: https://en.wikipedia.org/wiki/Linear-feedback_shift_register), which allows us to create a lightweight, yet a powerful random function.
- Thereby all these changes brought powerful execution time improvements when executed both in the CSUG server and on my Laptop. And all code has been verified to run without errors by adding the "-v" argument in all cases.


Results:
In this section I will only show the "execution time taken by the malloc()+Image Generation [ImageInitialize()] portion of the program", in all the cases since that was the only portion of the code that I modified (since it was the only portion that was not parallelized) and contains all the improvements I made:
(The remaining portion of the code takes the same time to execute in both optimised & unoptimised version for a given image size & no. of threads, and hence not included in this result)

Note: All tests are done on an image size of "32768 x 32768"

1) Execution times on the server-1 machine:

A) Unoptimised Code (can run only on 1 thread): 53.41 seconds
 
B) Optimised Code (can run in multiple threads):
    (i)    1 thread : 38.49 seconds	(This improvement is due to the optimisation of the rand() function)
    (ii)   2 threads: 19.48 seconds
    (iii)  4 threads:  9.98 seconds
    (iv)   8 threads:  5.31 seconds
    (v)   16 threads:  2.89 seconds
    (vi)  32 threads:  2.03 seconds
    (vii) 64 threads:  1.81 seconds	(Note: 64 was the highest number of cores in this server)


2) Execution times on node2x14a.csug.rochester.edu server:

A) Unoptimised Code (can run only on 1 thread): 50.68 seconds
 
B) Optimised Code (can run in multiple threads):
    (i)    1 thread : 39.19 seconds
    (ii)   2 threads: 19.95 seconds
    (iii)  4 threads: 11.82 seconds
    (iv)   8 threads:  6.82 seconds
    (v)   16 threads:  3.92 seconds
    (vi)  32 threads:  2.98 seconds
    (vii) 56 threads:  2.50 seconds		(Note: 56 was the highest number of cores in this server)

Therfore the performance improvement in the non-parallelized part of the code, are clearly observable from the above results after parallelizing.

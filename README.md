
Problem Statement: 

Scalable, flexible, efficient C code for parsing a large binary file containing a set of tuples and corresponding scores, and creating text output files containing subsets of tuples in the required format. 

Description: 

The input file is a set of unordered tuples, each with a corresponding score. A tuple is a vector of integers >= 0, with no repeating integer. Each tuple also has a floating point score that can be positive or negative. 

The executable must take the following as inputs.

Input: 

- 2 binary files, format described below
- 1 unsigned int n (4 bytes, 0 <= n <= 1,000,000)
- 1 unsigned int k (4 bytes, 0 <= k <= 100,000)
- 1 double b (8 bytes, 0 <= b <= 1)

Optional:

- 1 double s (8 bytes, -100,000 <= s <= 100,000)

Output:

The software should produce be 4 or 5 ascii text files, as described below. 

Input File Specifications:

Input file 1:
This binary file gives information about the main binary file. It is exactly 28 bytes.

type          | bytes       | field     | description
-----------------------------------------------------
signed int    | 4           | 0         | zero
signed int    | 4           | d         | dimension (# ints in each tuple) 
signed int    | 4           | n_vars    | number of variables total
unsigned long | 8           | n_tups    | number of tuples
double        | 8           | avg       | average of all scores
----------------------------------------------------------------
              | 28 bytes total

Input file 2:
This is the main input file. It's size is determined by the values in the first binary file.

type          | bytes       | field     | description
-----------------------------------------------------
int[]         | d*4         | t1        | tuple 1
double        | 8           | t1_s      | tuple 1 score
int[]         | d*4         | t2        | tuple 2
double        | 8           | t2_s      | tuple 2 score
...            ...            ...        ...
int[]         | d*4         | tlast     | last tuple
double        | 8           | tlast_s   | last tuple score
-----------------------------------------------------------
              | n_tups*((4*d)+8) bytes total

Every int in a tuple vector will be in the range [0,n_vars-1]. There will be no duplicate values in a tuple, which means n_tups <= nCr(n_vars, d). There is no ordering of the values within a tuple, and no ordering across all tuples. The dimension of each tuple will be between 2 and 8. (2 <= d <= 8). 

Output File Specifications:

All floating point values in the output files should have 10 significant digits beyond the decimal point.

out1.txt: top tuples sorted

Text file containing n tuples with the highest score, sorted by maximum score. If n=0, this file is empty.

out2.txt: bottom tuples sorted

Similar to out1.txt. Text file containing n tuples, with the lowest score, sorted by minimum score. If n=0, this file is empty.

For out1.txt and out2.txt, tuples are printed 1 per line, values separated by a tab, and the score of the tuple at the end with 10 digits beyond the decimal points. For example, a tuple of dimension 3, with the variables 4 5 and 6 and a score of 0.01 would look like:

4    5    6    0.0100000000

out3.txt: sorted tuples by variable

This file contains at most (2*k) + 1 lines for each value (variable) that appears in at least 1 tuple. The first line contains the variable number, the average of scores for all tuples that contain the variable, all separated by a tab. The next k lines are the highest scoring tuples that contain the variable , sorted by maximal score. The next k lines are the lowest scoring tuples that contain the variable, sorted by minimal score. If there are fewer than k tuples that contain the value, print all of the tuples that contain the value.

out4.txt: histogram

The first line of this file will be the minimum and maximum score in the file, respectively, separated by a tab. If b > 0, there will be floor((max-min)/b)+1 lines following the first line. Each line has a count of how many scores fall in the range determined by the bin size b, starting at the minimum value. So line 2 should have a count of scores in the range [min, b+min), line 3 has count of scores in [min+b, (2*b)+min), etc.

For example, if b=0.01, and the scores in the input file are 0.0100, 0.0120, 0.0299, 0.0300, 0.0310, 0.060, 0.100 then the output file should have 11 lines total and look like.

0.0100000000    0.1000000000
2
1
2
0
0
1
0
0
0
1

The sum of the numbers in each line (excluding first line) should equal total number of tuples in the input file (n_tups).
Note that there are 10 bins: [0.01, 0.02), [0.02, 0.03), ..., [0.09, 0.10), [0.10, 0.11). There is a need for the 10th bin in order to include the border value 0.100, which is not included in the 9th bin.

out5.txt: extra, unsorted tuples and distance from avg score

This file is only created if the value s is passed. 

This file will contain all tuples and corresponding scores above avg + (s*std) if dimension d is even, or below avg-(s*std) if dimension d is odd. The format of each tuple will look like out1.txt and out2.txt, except there will be an extra column (separated by tab) indicating how many standard deviations away from the mean the tuple score is.

A few things to note:

- avg is already calculated and given in the first binary file.

- This file will require calculating the standard deviation of all scores beforehand.

- These tuples do NOT need to be sorted by their score.

Command Line options:

- The software should use the command line options as used in the demo application:

Usage:   run [options] <input1> <input2>
Options:
         -n INT    number of sorted tuples for out1.txt and out2.txt
         -k INT    number of sorted tuples for out3.txt
         -b FLOAT  bin width for out4.txt
         -s FLOAT  number of standard deviations for out5.txt
 
For threads we could have such option:
 
         -t INT  number of threads

If you use threads, you should choose what you consider the best configuration, and use it as the default configuration if the -t command line option is not used. Please mention in the documentation which is the default value. If you do use threads, you may suggest some other option you think might be better, and we'll test with that option also.
For performance testing we will use big values for the -n and -k options (the maximum values, or close to the maximum).
For -b and -s options, we will choose some values that will produce files containing a number of records equal to 25%-75% of the number of tuples from the input file.
 
Other Requirements:

- The software should only pass through the input file 1 time, or 2 times if the optional value s is passed.

- The software can keep all tuples and scores needed for output files in RAM (determined by k and n), and should not use more than 4GB of RAM on top of this. If k=0, n=0, and b=0, the program should not use more than 4GB RAM.  We will be scoring the submissions based on the efficiency of their RAM usage compared to each other.  The weighting for RAM Usage is 25% of the total score.   
Note: Valgrind compatibility is a requirement.   We will test it with the command below and will take into consideration the maximum live memory. If the test crashes and we get no result, the score will be 1.
    valgrind --tool=exp-dhat <command>

- Parallelization is OK using pthreads.

- The test system architecture is x86_64 GNU/Linux.    All the solutions will be executed on a common AWS m4.large 14.04 Ubuntu with 8 GB RAM and provisioned IOPS.

- You must provide all source files, along with Makefile or other instructions for compilation and execution.

- No external libraries can be used, except for the C standard libraries , pthread library, and other POSIX compliant functions.

- The software will be judged on speed for various input files and values of n,k,b. The program should run fastest when n,k, and b are 0.  The submissions will be compared to each other to generate a execution speed score.  The weighting for this execution speed element is 25%.  Please review the scorecard to understand the details of the performance scoring elements.

Source code and Sample input files:

Here is a link to the existing source code and sample input files that we will use for testing:
https://drive.google.com/open?id=0BycYpuMUQwQwdnRtSFBjcFIzcnM

The source code handles the command line arguments and provides an implementation for reading the input files and producing file out5.txt.
You need to implement the rest of the functionality, and you are encouraged to try to improve the existing code, because all the functionality will count during testing.
PLATFORMS

Linux
TECHNOLOGIES

C Data Science
Final Submission Guidelines
- Please submit all your source code and documentation in a zip format.  There is existing code provided as a starting point for this challenge but the use of this code is optional provided your code maintains the interfaces defined above.

- You should submit a Makefile to build your application. An existing Makefile has been provided as a starting point.

- The previously written code for this application can be found in the Code Documents forums attached to this challenge.  Instructions on how to execute the existing code are provided in the readme.txt


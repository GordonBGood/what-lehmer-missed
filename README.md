# What D.H. Lehmer Missed

## Introduction

This repository is a demonstration of what D.H. Lehmer missed in his 1959 implementation of a combinational prime counting algorithm to count the number of primes up to 10 billion (10<sup>10</sup>).  His work was based on that of Ernst Meissel of between the years 1870 to 1886 where Meissel was able to hand calculate the number of primes up to one billion (10<sup>9</sup>), which work was in turn based on Adrien-Marie Legendre's inclusion/exclusion principle of 1798 leading up to Legendre's calculation of the number of primes to a million (10<sup>6</sup>) in 1808.  What will be demonstrated here that Lehmer missed is the implementation of these prime counting functions using "Partial Sieving", which Meissel certainly used in being able to hand calculate the number of primes to a billion (10<sup>9</sup>) within his lifetime, as the about 2.7 million long divisions required not using "Partial Sieving" would have required almost 90 years of calculating about 100 long divisions almost every day.  In hand calculating the number of primes to a million (10<sup>6</sup>), Legendre may or may not have used "Partial Sieving", as the about 15,321 long divisions required (odds-only) not using it would have easly been possible within the about seven years in which he computed it although it would have been much easier with the reduction to only 2,908 long divisions enabled by using "Partial Sieving".  This repository demonstrates the amount of work required to compute these values and the implementations that could have been used to accomplish them, also showing that Lehmer could have implemented these algorithms on the IBM 701 mainframe computer which he used.

## The Legendre Prime Counting Algorithm

$$
\begin{align*}
\pi(x) &= \phi(x,a) + a - 1 \\
a &= \pi(\sqrt{x}) \\
\phi(x,a) &= {x} - \sum_{i}^{a} {\lfloor \frac{x}{p_i} \rfloor}
                 + \sum_{j>i}^{a} {\lfloor \frac{x}{p_i p_j} \rfloor}
                 - \sum_{k>j>i}^{a} {\lfloor \frac{x}{p_i p_j p_k} \rfloor}
                     \dots + (-1)^a {\lfloor \frac{x}{p_1 p_2 p_3 \dots p_a} \rfloor}  \\
&= \sum_{d \mid P_a} \mu(d) {\lfloor \frac{x}{d} \rfloor} \\
&= \phi(x,a-1) - \phi(x/p_a, a-1) \\
P_a &= \prod_{i=1}^{a} p_i \\
\end{align*} 
$$

Understand that $$P_a$$ is the product of all of the primes up to the square root of the counting limit.  $\mu(d)$ is the Moebius Function for ${d}$ which normally has values of plus or minus one and zero but as zero represents any value that has a factor of a prime squared and $$P_a$$ contains only unique prime factors by definition, a zero result in this case is impossible.  A plus one Moebius value means that the argument has an odd number of prime factors and a minus one value means that the argument has an even number of prime factors, so application here means that the summation adds the floor of the quotients when the number of prime factors is even and subtracts when odd.  Legendre would not have known this function by name as the name is relatively new, but it is clear from the orginal $\phi(x,a)$ expansion that odd numbers of factors are subtracted and even numbers are added.  The notation ${d \mid P_a}$ means that ${d}$ is in the set of all numbers that can be evenly divided into $P_a$, meaning they are all the combinations of products of unique primes up to the square root of ${x}$.  This Moebius Function expressed summation for the summation of the inclusion/exclusion primciple above and it is likely what Legendre would have used to hand calculate the number of primes to a million for a couple of reasons:  1) the idea of function recursion hadn't been invented yet, nor was there a concept of a last-in/first-out stack as required by recursive functions, and 2) this Moebius Function summation can have a much better asymptotic complexity of `O(x**(2/3)/((log x)**2))` than the `O(x/((log x)**2))` of the recursive formulation if combined with "Partial Sieving" or doing processing after each culling pass by each individual base prime

The reason that "Partial Sieving" can exponentially reduce the required number of operations is due to that its formulation initializes to all the values that are products of the primes and gradually reduces them by eliminating the ones that are factors of the base primes in turn starting at the least as compared with starting with all the individual base primes and building up all the combinations of the products (which requires many more recursions) for the recursive formulation.  As can be seen in the discussion of Lehmer's work below, many modern programmers become fixated on the idea of using recursion to calculate the $$\phi(x,n)$$ value, ignoring that this has exponentially worse performance complexity than by using "Partial Sieving" as explained above.  Legendre would have had less tendency to fall into this trap as the idea of function recursion hadn't really been investigated yet, but may have fallen into it anyway due to the easily derivable recursive expression as above.

## The Meissel Prime Counting Algorithm

$$
\begin{align*}
\pi(x) &= \phi(x,a) + a - 1 - P2(x) \\
a &= \pi(\sqrt[3]{x}) \\
\phi(x,a) &= {x} - \sum_{i}^{a} {\lfloor \frac{x}{p_i} \rfloor}
                 + \sum_{j>i}^{a} {\lfloor \frac{x}{p_i p_j} \rfloor}
                 - \sum_{k>j>i}^{a} {\lfloor \frac{x}{p_i p_j p_k} \rfloor}
                     \dots + (-1)^a {\lfloor \frac{x}{p_1 p_2 p_3 \dots p_a} \rfloor}  \\
&= \sum_{d \mid P_a} \mu(d) {\lfloor \frac{x}{d} \rfloor} \\
&= \phi(x,a-1) - \phi(x/p_a, a-1) \\
P_a &= \prod_{i=1}^{a} p_i \\
P2(x) &= \sum_{i>a}^{b} \pi({\lfloor \frac{x}{p_i} \rfloor}) - \pi(p_i) + 1 \\
b &= \pi(\sqrt{x}) \\
\end{align*} 
$$

From the above formula, it is easy to see the similarities to the Legendre formula with the differences just that the Meissel formula only requires the calculation of $$\phi(x,\pi(\sqrt[3]{x}))$$ instead of $$\phi(x,\pi(\sqrt{x}))$$ but at the cost of having to sieve to about x<sup>(2/3)</sup> instead of $$\sqrt{x}$$ and also compute the `P2` compensation value, although given the results of the sieve buffer that is quite a simple calculation requiring not many division operations.  Although the computation of the "Phi" value for the Meissel formula is already easier than for the Legendre furmula, it also benefits exponentially by the use of "Partial Sieving" just as a Legendre implmentationd does.

## The Lehmer Prime Counting Algorithm

$$
\begin{align*}
\pi(x) &= \phi(x,a) + a - 1 - P2(x) - P3(x) \\
a &= \pi(\sqrt[3]{x}) \\
\phi(x,a) &= {x} - \sum_{i}^{a} {\lfloor \frac{x}{p_i} \rfloor}
                 + \sum_{j>i}^{a} {\lfloor \frac{x}{p_i p_j} \rfloor}
                 - \sum_{k>j>i}^{a} {\lfloor \frac{x}{p_i p_j p_k} \rfloor}
                     \dots + (-1)^a {\lfloor \frac{x}{p_1 p_2 p_3 \dots p_a} \rfloor}  \\
&= \sum_{d \mid P_a} \mu(d) {\lfloor \frac{x}{d} \rfloor} \\
&= \phi(x,a-1) - \phi(x/p_a, a-1) \\
P_a &= \prod_{i=1}^{a} p_i \\
P2(x) &= \sum_{i>a}^{b} \pi({\lfloor \frac{x}{p_i} \rfloor}) - \pi(p_i) + 1 \\
b &= \pi(\sqrt{x}) \\
P3(x) &= \sum_{i>c}^{a} \sum_{j>i}^{b_i} \pi({\lfloor \frac{x}{p_i p_j} \rfloor}) - \pi(p_j) + 1 \\
b_i &=  \pi({\lfloor \frac{x}{p_i} \rfloor}) \\
c &= \pi(\sqrt[4]{x}) \\
\end{align*} 
$$

The above Lehmer prime counting formula can be seen to be just an extension above the Meissel formula just as the Meissel is an extension of the Legendre forumula except that "Phi" is now $$\phi(x,\pi(\sqrt[4]{x}))$$ instead of $$\phi(x,\pi(\sqrt[3]{x}))$$, thereby reducing the number of long division operations necessary to compute "Phi" at the cost of having to sieve to about x<sup>(3/4)</sup> instead of x<sup>(2/3)</sup> plus a somewhat more complex "P3" correction in addition to the "P2" correction.  Lehmer likely wasn't too concerned with the execution time cost of sieving as he sieved a sufficiently large range of fifty million once and saved the sieved bit stream to magnetic tape(s), then used that sieved stream in all subsequent computations of ranges of prime counts.  He almost certainly chose this extended prime counting formula in order to reduce the number of long division operations required for each range, especially for counting the number of primes to 10 billion (10<sup>10</sup>) which would have required about 25 million divisions for the odds-only non "Partial Sieving" Meissel formula to about 3.2 million for the odds-only non "Partial Sieving" Lehmer forumula.  In order to further reduce the number of long division operations in counting the number of primes to ten billion, he also used a "Tiny Phi" Look Up Table (LUT) for the base primes of two, three, five, seven, and eleven (for the first five base primes) that would reduce the number of long division operations to about 7.7 million and about 900 thousand, respectively, for the Meissel and Lehmer formulas, spending some development time in compressing the required LUT into the limited available memory.

While the above reduction in the required number of long division operations seems impressive, what Lehmer missed was the method of "Partial Sieving" that had already been used by Meissel in that, even without the "Tiny Phi" LUT, by using it he could have reduced the number of long divison operations to only about 63,000 to compute the number of primes up to ten billion, even using the Meisel formula, with the Meissel formula not requiring the somewhat complex "P3" computation and only requiring sieving to the lower limit of less than five million rather than at about 33 million as for the Lehmer formula.  While using the "Partial Sieving" method to compute "Phi" is somewhat more complex than using the recursive method he used, an implementation would have fit into the limited memory of the IBM 701 mainframe computer he used as the "Tiny Phi" LUT reduction isn't necessary, as I will show in the next section.

## An IBM 701 Implementation of the Meissel Algorithm Using Partial Sieving

There is a simulator for the IBM 701 available in [the open source Open SIMH repository](https://github.com/open-simh/simh) although it wasn't fully developed and my Pull Request(s) still hasn't been merged.  My bug fixing patch [forked version of this repository with the needed fixes](https://github.com/GordonBGood/sims) can be used to generate an executable for the "i701" simulator by installing the required pre-requisites and running "make i701" from within a clone of the main folder of the repository, with the "i701" executable generated in a sub "BIN" directory/folder.  This simulator can be throttled to run at the about 83,000 machine cycles per second that the original mainframe computer ran, which is thousands of times slower than if run unthrottled on a modern personal computer.  Although the simulator includes a mini machine code assembler and disassembler, it is not symbolic so it would be very difficult to write and debug a project of this size using hand assembled code, so I wrote a Python symbolic cross-assembler for the IBM 701 and have included it in this repository.  It can be used as "`python asmi701.py <path to the IBM 701 assembly source code>`" and will generate both "*.oct" and "*.cdr" files that can be loaded into the simulator much as if reading from Hollerith cards in the original computer.  There are batch "config" files provided so that a computer run can be made by just running "`i701 <path to config file>`", with the input argument, the resulting prime count, and timing statistics shown for the run.  This configuration file can be edited with a text editor to change the input parameters to various values to count the primes to a range as high as 34,359,738,367 (2<sup>35</sup>-1) in about an hour at throttled "real" speed.

---
title: Code-makeover on how to code for better efficiency
teaching: 15
exercises: 0
questions:
- How to write the most efficient code?
objectives:
- Learn good tips and tools to improve your code.
keypoints:
- CPU, memory, and build time optimizations are possible when good code practices are followed.
---

## DUNE Computing Training May 2021

<!--
## Session Video

<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/k74QnmovOzA" title="DUNE Computing Tutorial May 2021 Code-makeover on how to code for better efficiency" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>-->

### Code Make-over

**How to improve your code for better efficiency**

DUNE simulation, reconstruction and analysis jobs take a lot of memory and CPU time.  This owes to the large size of the Far Detector modules as well as the many channels in the Near Detectors.  Reading out a large volume for a long time with high granularity creates a lot of data that needs to be stored and processed.

### CPU optimization:

**Run  with the “prof” build when launching big jobs.**  While both the "debug" and "prof" builds have debugging and profiling information included in the executables and shared libraries, the "prof" build has a high level of compiler optimization turned on while "debug" has optimizations disabled.  Debugging with the "prof" build can be done, but it is more difficult because operations can be reordered and variables put in CPU registers instead of inspectable memory. The “debug” builds are generally much slower, by a factor of four or more.  Often this difference is so stark that the time spent repeatedly waiting for a slow program to chug through the first trigger record in an interactive debugging session is more costly than the inconvenience of not being able to see some of the variables in the debugger.  If you are not debugging, then there really is (almost) no reason to use the “debug” builds.  If your program produces a different result when run with the debug build and the prof build (and it’s not just the random seed), then there is a bug to be investigated.

**Compile your interactive ROOT scripts instead of running them in the interpreter**   At the ROOT prompt, use .L myprogram.C++ (even though its filename is myprogram.C).  Also .x myprogram.C++ will compile and then execute it.  This will force a compile.  .L myprogram.C+ will compile it only if necessary.

**Run gprof or other profilers like valgrind's callgrind:**   You might be surprised at what is actually taking all the time in your program.  There is abundant documentation on the [web][gnu-manuals-gprof], and also the valgrind online documentation.
There is no reason to profile a "debug" build and there is no need to hand-optimize something the compiler will optimize anyway, and which may even hurt the optimality of the compiler-optimized version.

**The Debugger can be used as a simple profiler:** If your program is horrendously slow (and/or it used to be fast), pausing it at any time is likely to pause it while it is doing its slow thing.  Run your program in the debugger, pause it when you think it is doing its slow thing (i.e. after initialization), and look at the call stack.  This technique can be handy because you can then inspect the values of variables that might give a clue if there’s a bug making your program slow.  (e.g. looping over 10<sup>15</sup> wires in the Far Detector, which would indicate a bug.**.

**Don't perform calculations or do file i/o that will only later  be ignored.**  It's just a waste of time.  If you need to pre-write some code because in future versions of your program the calculation is not ignored, comment it out, or put a test around it so it doesn't get executed when it is not needed.


**Extract constant calculations out of loops.**


<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;">

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;border-left-width: thick; border-left-color: #280071;border-radius: 5px;">Code Example (BAD)</div>

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (GOOD)</div>

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px; border-left-width: thick; border-left-color: #280071;border-radius: 5px;">
double sum = 0;<br>
for (size_t i=0; i<n_channels; ++i)<br>
{<br>
  sum += result.at(i)/TMath::Sqrt(2.0);<br>
}<br>
</div>
<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double sum = 0;<br>
double f = TMath::Sqrt(0.5);<br>
for (size_t i=0; i<n_channels; ++i)<br>
{<br>
  sum += result.at(i)*f;<br>
}<br>
</div>

</div><!--side by side table by DeMuth-->


The example above also takes advantage of the fact that floating-point multiplies generally have significantly less latency than floating-point divides (still true, even with modern CPU.

**Use sqrt():** Don’t use `pow()` or `TMath::Power` when a multiplication or `sqrt()` function can be used.

<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;">

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;border-left-width: thick; border-left-color: #280071;border-radius: 5px;">Code Example (BAD)</div>

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (GOOD)</div>
<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px; border-left-width: thick; border-left-color: #280071;border-radius: 5px;">
double r = TMath::Power(  TMath::Power(x,2) + TMath::Power(y,2), 0.5);
</div>

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double r = TMath::Sqrt( x*x + y*y );
</div>
</div>

The reason is that `TMath::Power` (or the C math library’s `pow()`) function must take the logarithm of one of its arguments, multiply it by the other argument, and exponentiate the result.  Modern CPUs have a built-in `SQRT` instruction.  Modern versions of `pow()` or `Power` may check the power argument for 2 and 0.5 and instead perform multiplies and `SQRT`, but don’t count on it.

If the things you are squaring above are complicated expressions, use `TMath::Sq()` to eliminate the need for typing them out twice or creating temporary variables.  Or worse, evaluating slow functions twice.  The optimizer cannot optimize the second call to that function because it may have side effects like printing something out to the screen or updating some internal variable and you may have intended for it to be called twice.


<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;">
<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;border-left-width: thick; border-left-color: #280071;border-radius: 5px;">Code Example (BAD)</div>

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (GOOD)</div>

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px; border-left-width: thick; border-left-color: #280071;border-radius: 5px;">
double r = TMath::Sqrt( slow_function_calculating_x()*slow_function_calculating_x() +  slow_function_calculating_y()*slow_function_calculating_y()  );
</div>

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double r = TMath::Sqrt( TMath::Sq(slow_function_calculating_x()) + TMath::Sq(slow_function_calculating_y()));
</div>
</div>

**Don't call `sqrt()` if you don’t have to.**

<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;">
<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;border-left-width: thick; border-left-color: #280071;border-radius: 5px;">Code Example (BAD)</div>

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (GOOD)</div>

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px; border-left-width: thick; border-left-color: #280071;border-radius: 5px;">
if (TMath::Sqrt( x*x + y*y ) < rcut )<br>
{<br>
  do_something();<br>
}
</div>

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double rcutsq = rcut*rcut;<br>
if (x*x + y*y < rcutsq)<br>
{<br>
  do_something();<br>
}
</div>
</div>

<!--
hopefully factor out of loops this line:

~~~
double rcutsq = rcut*rcut;
~~~
{: .source}

and then execute:

~~~
if ( x*x + y*y < rcutsq)  
{  
  do_something();  
}  
~~~
{: .source}
-->

**Use binary search features in the STL rather than a step-by-step lookup.**

~~~
std::vector<int> my_vector;
(fill my_vector with stuff)

size_t indexfound = 0;
bool found = false;
for (size_t i=0; i<my_vector.size(); ++i)
{
  if (my_vector.at(i) == desired_value)
    {
	indexfound = i;
	found = true;
    }
}
~~~
{: .source}

If you have to search through a list of items many times, it is best to sort it and use std::lower_bound; see the example [here][cpp-lower-bound].
`std::map` is sorted, and `std::unordered_map` uses a quicker hash table.  Generally looking things up in maps is `O(log(n))` and in a `std::unordered_map` is `O(1)` in CPU time, while searching for it from the beginning is `O(n)`.  The bad example above can be sped up by an average factor of 2 by putting a break statement after `found=true;` if you want to find the first instance of an object.  If you want to find the last instance, just count backwards and stop at the first one you find; or use `std::upper_bound`.

**Don’t needlessly mix floats and doubles.**


<div style="display: grid;grid-template-columns: repeat(2,460px);grip-gap: 5px;width:1120px;border: 2px solid #ffffff;font-family:Courier, monospace;color: #000000;font-size: 10pt;">
<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;border-left-width: thick; border-left-color: #280071;border-radius: 5px;">Code Example (BAD)</div>

<div style="background-color: #EFEAF4; border: 1px solid #000000;text-align: center; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;color: #280071;">Code Example (GOOD)</div>

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px; border-left-width: thick; border-left-color: #280071;border-radius: 5px;">
double sum = 0;<br>
std::vector &lt;double&gt; results;<br>
(fill lots of results)<br>
for (size_t i=0; i<results.size(); ++i)<br>
{<br>
  float rsq = results.at(i)*result.at(i);<br>
  sum += rsq;<br>
}
</div>

<div style="background-color: #EBEBEB; border: 1px solid #000000;text-align: left; padding-left: 10px;padding-right: 10px;padding-top: 5px;padding-bottom: 5px;">
double sum = 0;<br>
std::vector &lt;double&gt; results;<br>
(fill lots of results)<br>
for (size_t i=0; i<results.size(); ++i)<br>
{<br>
  sum += TMath::Sq(results.at(i));<br>
}
</div>
</div>

**Check for NaN and Inf.**  While your program will still function if an intermediate result is `NaN` or `Inf` (and it may even produce valid output, especially if the `NaN` or `Inf` is irrelevant), processing `NaN`s and `Inf`s is slower than processing valid numbers.  Letting a `NaN` or an `Inf` propagate through your calculations is almost never the right thing to do - check functions for domain validity (square roots of negative numbers, logarithms of zero or negative numbers, divide by zero, etc.) when you execute them and decide at that point what to do.  If you have a lengthy computation and the end result is `NaN`, it is often ambiguous at what stage the computation failed.

**Minimize cloning TH1’s.**  It is really slow.

**Minimize formatted I/O.**   Formatting strings for output is CPU-consuming, even if they are never printed to the screen or output to your logfile.  `MF_LOG_INFO` calls for example must prepare the string for printing even if it is configured not to output it.

**Use sparse matrix tools where appropriate.**  This also saves memory.

**Minimize database access operations.**  Bundle the queries together in blocks if possible.  Do not pull more information than is needed out of the database.  Cache results so you don’t have to repeat the same data retrieval operation.

Use `std::vector::reserve()` in order to size your vector right if you know in advance how big it will be.  `std::vector()` will, if you `push_back()` to expand it beyond its current size in memory, allocate twice the memory of the existing vector and copy the contents of the old vector to the new memory.  This operation will be repeated each time you start with a zero-size vector and push_back a lot of data.
Factorize your program into parts that do i/o and compute.   That way, if you don’t need to do one of them, you can switch it off without having to rewrite everything.  Example:  Say you read data in from a file and make a histogram that you are sometimes interested in looking at but usually not.  The data reader should not always make the histogram by default but it should be put in a separate module which can be steered with fcl so the computations needed to calculate the items to fill the histogram can be saved.

## Memory optimization:

Use `valgrind`.  Its default operation checks for memory leaks and invalid accesses.  Search the output for the words “invalid” and “lost”.  Valgrind is a `UPS` product you can set up along with everything else.

~~~
setup valgrind
valgrind --leak-check=yes myprog arg1 arg2
~~~
{: .source}

More information is available [here][valgrind-quickstart]

Use `massif`.  `massif` is a heap checker, a tool provided with `valgrind`; see documentation [here][valgrind-ms-manual].

**Free up memory after use.**  Don’t hoard it after your module’s exited.

**Don’t constantly re-allocate memory if you know you’re going to use it again right away.**

**Use STL containers instead of fixed-size arrays, to allow for  growth in size.**   Back in the bad old days (Fortran 77 and earlier), fixed-size arrays had to be declared at compile time that were as big as they possibly could be, both wasting memory on average and creating artificial cutoffs on the sizes of problems that could be handled.  This behavior is very easy to replicate in C++.  Don’t do it.

**Be familiar with the structure and access idioms.** These include `std::vector`, `std::map`, `std::unordered_map`, `std::set`, `std::list`.

**Minimize the use of new and delete to reduce the chances of memory leaks.**  If your program doesn’t leak memory now, that’s great, but years from now after maintenance has been transferred, someone might introduce a memory leak.

**Use move semantics to transfer data ownership without copying it.**

**Do not store an entire event’s worth of raw digits in memory all at once.**  Find some way to process the data in pieces.

**Consider using more compact representations in memory.**  A `float` takes half the space of a double.  A `size_t` is 64 bits long (usually).  Often that’s needed, but sometimes it’s overkill.

**Optimize the big uses and don’t spend a lot of time on things that don’t matter.**  If you have one instance of a loop counter that’s a `size_t` and it loops over a million vector entries, each of which is an `int`, look at the entries of the vector, not the loop counter (which ought to be on the stack anyway).

**Rebin histograms.**  Some histograms, say binned in channels <i>x</i> ticks or channels <i>x</i> frequency bins for a 2D FFT plot, can get very memory hungry.

## Build time optimization:

**Minimize the number of #included files.**  If you don’t need an #include, don’t use it.  It takes time to find these files in the search path and include them.

**Break up very large source files into pieces.**   `g++’s` analysis and optimization steps take an amount of time that grows faster than linearly with the number of source lines.

## Workflow optimization:

**Pre-stage your datasets**  It takes a lot of time to wait for a tape (sometimes hours!).  CPUs are accounted by wall-clock time, whether you're using them or not.  So if your jobs are waiting for data, they will run slowly even if you optimized the CPU usage.  Pre-stage your data!

**Run a test job**  If you have a bug, you will save time by not submitting large numbers of jobs that might not work.

**Write out your variables in your own analysis ntuples (TTrees)**  You will likely have to run over the same MC and data events repeatedly, and the faster this is the better. You will have to adjust your cuts, tune your algorithms, estimate systematic uncertainties, train your deep-learning functions, debug your program, and tweak the appearance of your plots.  Ideally, if the data you need to do these operatios is available interctively, you will be able to perform these tasks faster.  Choose a minimal set of variables to put in your ntuples to save on storage space.

**Write out histograms to ROOTfiles and decorate them in a separate script**  You may need to experiment many times with borders, spacing, ticks, fonts, colors, line widths, shading, labels, titles, legends, axis ranges, etc.  Best not to have to re-compute the contents when you're doing this, so save the histograms to a file first and read it in to touch it up for presentation.



[cpp-lower-bound]: https://en.cppreference.com/w/cpp/algorithm/lower_bound
[gnu-manuals-gprof]: https://ftp.gnu.org/old-gnu/Manuals/gprof-2.9.1/html_mono/gprof.html
[valgrind-quickstart]: https://www.valgrind.org/docs/manual/quick-start.html
[valgrind-ms-manual]: https://www.valgrind.org/docs/manual/ms-manual.html


{%include links.md%}

# Functional Programming with C and Make
Or, how we tackled tomorrow's problem by yesterday's computer.


## 1. The Problem

We needed do certain processes on a massive data set. The data were
pointclouds (collection of X-Y-Z values) of the shape of the Step
Pyramid in Saqqara, Egypt. Number of the points in the pointclouds was
over ___. The data were obtained by specially designed shape sensor
called Zoser.

The data were stored originally in binary format and were converted in
text format. The data files were separated in 100 to 1,000 files and
were not integrated intentionally for portability reason.

## 2. Prequalify

The data must have passed a prequalify. The data may have contained
obvious errors, e.g. lack of fields. We started our processing with
filtering out those trivial errors.

Since the data were so huge that we could not store all of them on
memory, we needed special care on our programming. Though we knew that
some Virtual Machine technology would help us, we went to an extreme
approach. We used AWK.

We wrote awkwardly simple AWK script that only checked the number of
fields of each lines in the data files. For-loop statement of Bash
greatly helped us to apply this single AWK script to multiple files.

## 3. The First Functional Approach

Soon we found that we could, and should, bring functional programming
paradigm in this field, or we could easily loose our way goal.

We changed the rule. The original data files had txt suffixes because
they were text files. We decided to give another suffix to filtered
data though they were still text files. We chosen pq. The prequalify
was done by the following rule on GNU make:

    %.pq: %.txt
    $(awk) -f prequalify.awk $? > $@

So was this a functional approach? Surely yes. First, no one made side
effects. Second, giving a same parameter to the system always caused
the same results.

The functional approach of filtering texts was not a new idea. Unix
hackers have preferred pipeline approach from the ice-age. We just
followed the hacker's way at this moment.

## 4. More Functional Approach

We divided our computing process into several stages. Dividing heavy
computing process into relatively light and small pieces has many
advantages. One of them is that we can follow the computation step by
step so that we can fix the program when we find an error.

Combining a couple of commands on Unix-like systems can be done
generally by these two different approaches.

1. batch process. For example, "procedure1 data; procedure2 data" or "procedure1 data && procedure2 data" does this.
2. pipeline. For example, "cat data | function1 | function2 > result"

By assistance of good virtual memory system (1) will work on massive
data; on the other hand by assistance of well-designed blocking
pipeline system (2) will work on massive data too.

We did not choose (1) nor (2). At this point the author does not
insist to say that our OS had poor virtual memory system or it had
inferior pipeline system. (To confess, we used a Mac OS X 10.5.) We
chosen the third way.

Each part of the computation processes created the intermediate result
files in our system. The intermediate result files had each different
suffixes. For example, the first stage would read raw txt data and
produce intermediate s1 (meaning result of the stage 1) file. The
second stage would read s1 and produce s2…

This approach had a couple of major advantages. One was (A) using
intermediate files was memory-friendly. Since we could not load full
data on memory, we needed to process data block by block. Only
accumulation variables and part of data were kept on memory. One of
disadvantages of this approach was obviously in file access. This was
something that we needed to pay for the advantages. (We stored all
intermediate data in binary format to reduce I/O costs. Our
hand-crafted data viewer that could directly open such binary files
supported us for debugging.)

Another advantage was (B) different suffixes of the file names in each
computation stage made us possible to fully use GNU make system for
running computer. This was almost a killer. We just used make only for
covering up our poor ability to remember command-line parameters at
the begining. However, we soon fond that everything was now
automatic. When we found an error at some stage, we fixed the program,
and ran the make. Boom! The program we just fixed was recompiled and
the whole computation was immediately restarted from the point where
we found the error. The intermediate results that seemed properly
computed were remained while suspicious results were deleted and
recomputed.

## 5. Special Bonus

Using GNU make brought us a special bonus. By giving -j option to
make, the processors were at full throttle. The -j option tells GNU
make to run independent computations and/or compilations
simultaneously if possible. Dependency of each intermediate results
are automatically taken in account.

Actually by giving -j option to make made the total computation faster
than plain make by 20% to 50% on a quad core computer.

## 6. Beyond the Functional Approach

On Day 1 of the project we planned to do a certain block of
computation by C++. However, we soon found that we'd had better
implement smaller computational blocks instead of originally planned,
rather complex, data processing. We implemented basic vector algebras
and several interpolation formulae. Some of them were simply wrappers
on GNU Scientific Library. We then combined these building blocks on
Makefile.

This approach was quite simple and quick yet powerful enough to tackle
our problem. At that moment we felt as if we coded in super rigid
functional programming language even though we were coding in C++.

## 7. So What?

We wanted to share that all of us could easily get advantage of
functional programming style with old and good tools we all had loved:
make and your familiar language like C.

We opened the hint we have learned. But sometimes hints are only
hints. Thus, we open the all source codes we have developed and
used. The code is now grouped in two packages and hosted on
http://sourceforge.net/ . The fundamental part of the code is called
Vector Stream, the specialized part is called the Zoser code.





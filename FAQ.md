# Frequently Asked Questions #

Here is a list of frequently asked questions about EJML.  Most of these questions have been asked and answered several times already.

## Why does EJML crash when I try to process a matrix with 1,000,000 by 1,000,000 elements, or some other very large matrix? ##

If you are working with large matrices first do a quick sanity check.  Ask yourself, how much memory is that matrix using and can my computer physically store it?  Compute the number of required gigabytes with the following equation:

```
  memory in gigabytes = (columns * rows*8)/(1024*1024*1024)
```

Now take the number and multiply it by 3 or 4 to take in account overhead/working memory and that's about how much memory your system will need to do anything useful.  This is true for ALL dense linear algebra libraries.  EJML is also limited by the size of a Java array, which can have at most 2^32 elements.  If you are lucky and the system is sparse (full of mostly zeros) there might be other libraries out there which can help you.

The other potentially fatal problem is that very large matrices are very slow to process. So even if you have enough RAM on your computer the time to compute the solution could well exceed the lifetime of a typical human.

## Will EJML work on Android? ##

Yes EJML has been used for quite some time on Android.  The library does include a tinny bit of swing code, which will not cause any problems as long as you do not call anything related to visualization.

If you are getting compilation errors complaining about the swing code when using EJML then you need to stop compiling EJML inside of your Android project.  Just grab the precompiled jar and it into the lib/ directly and the Android development platform will take care of the rest.

In the past there was a nogui jar for Android.  That jar has been deprecated since it wasn't necessary.  Just use the standard jar.

## Multi-Threaded ##

Currently EJML is entirely single threaded.  The plan is to max out single threaded performance by finishing block algorithms implementations, then declare the library to be at version 1.0.  After that has happened, start work on multi-threaded implementations.  However, there is no schedule in place for when all this will happen.

The main driving factor for when major new features are added is when I personally need such a feature.  I'm starting to work on larger scale machine learning problems, so there might be a need soon.  Another way to speed up the process is to volunteer your time and help develop it.

## Sparse Matrix Support ##

EJML's current focus is on dense matrices, but could be extended in the future to support sparse matrices.  In the mean time the following libraries do provide some support for sparse matrices.  Note: I have not used any of these libraries personally.  Comment below with your experiences.

  * [CSparseJ](https://sites.google.com/site/piotrwendykier/software/csparsej)
  * [la4j](http://la4j.org/)
  * [MTJ](https://github.com/fommil/matrix-toolkits-java)
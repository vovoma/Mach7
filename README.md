<a href="https://github.com/solodon4/Mach7/blob/master/media/posters/OpenPatternMatching-OOPSLA-1.pdf?raw=true">
  <img src="https://raw.githubusercontent.com/solodon4/Mach7/master/media/posters/OpenPatternMatching-OOPSLA%20(1280x989).jpg" width="100%">
</a>

Mach7: Pattern Matching for C++
===============================

by [Yuriy Solodkyy](http://parasol.tamu.edu/~yuriys/), [Gabriel Dos Reis](http://parasol.tamu.edu/~gdr/), [Bjarne Stroustrup](http://parasol.tamu.edu/~bs/)

Abstract
--------

Pattern matching is an abstraction mechanism that can greatly simplify source code.
Commonly, pattern matching is built into a language to provide better syntax, faster 
code, correctness guarantees and improved diagnostics. Mach7 is a library solution 
to pattern matching in C++ that maintains many of these features. All the patterns 
in Mach7 are user-definable, can be stored in variables, passed among functions, 
and allow the use of open class hierarchies.

Mach7 by Example
----------------

Fibonacci numbers demonstrates [the use of patterns with built-in types](https://github.com/solodon4/Mach7/blob/master/code/numbers.cpp#L202-L216) in Mach7:

```C++
// Fibonacci numbers
int fib(int n)
{
    var<int> m;

    Match(n)
    {
      Case(1)     return 1;
      Case(2)     return 1;
      Case(2*m)   return sqr(fib(m+1)) - sqr(fib(m-1));
      Case(2*m+1) return sqr(fib(m+1)) + sqr(fib(m));
    }
    EndMatch
}
```

Lambda calculator demonstrates [use of pattern matching to decompose objects and nested patterns](https://github.com/solodon4/Mach7/blob/master/code/lambda.cpp#L97-L114):

```C++
// Lambda calculator
struct Term       { virtual ~Term() {}     };
struct Var : Term { std::string name;      };
struct Abs : Term { Var&  var;  Term& body;};
struct App : Term { Term& func; Term& arg; };

Term* eval(Term* t)
{
    var<const Var&> v; 
    var<const Term&> b,a;

    Match(*t)
    {
      Case(C<Var>())               return &match0;
      Case(C<Abs>())               return &match0;
      Case(C<App>(C<Abs>(v,b),a))  return eval(subs(b,v,a));
      Otherwise() cerr << "error"; return nullptr ;
    } 
    EndMatch
}
```

It can also be used to demonstrate [relational matching on several arguments](https://github.com/solodon4/Mach7/blob/master/code/lambda.cpp#L123-L140):

```C++
bool operator==(const Term& left, const Term& right)
{
    var<std::string> s;
    var<const Term&> v,t,f;

    Match(left,right)
    {
      Case(C<Var>(s),     C<Var>(+s)     ) return true;
      Case(C<Abs>(&v,&t), C<Abs>(&+v,&+t)) return true;
      Case(C<App>(&f,&t), C<App>(&+f,&+t)) return true;
      Otherwise()                          return false;
    }
    EndMatch

    return false; // To prevent all control path warning
}
```

Next example demonstrates that [the library can deal efficiently and in a type-safe manner with non-polymorphic classes](https://github.com/solodon4/Mach7/blob/master/code/xtl.cpp#L338-L361)
like boost::variant as well.

```C++
void print(const boost::variant<double,float,int>& v)
{
    var<double> d; var<float> f; var<int> n;

    Match(v)
    {
      Case(C<double>(d)) cout << "double " << d << endl; break;
      Case(C<float> (f)) cout << "float  " << f << endl; break;
      Case(C<int>   (n)) cout << "int    " << n << endl; break;
    }
    EndMatch
}
```

Breve syntax is not the only thing Mach7 has to offer - the generated code is 
[faster than Visitors](https://parasol.tamu.edu/~yuriys/posters/Mach7.pdf)!

For a more detailed set of examples, have a look at the code that was prepared for 
[CppCon 2014 presentation](http://bit.ly/AcceptNoVisitors),
and implemented using [visitors](https://github.com/solodon4/Mach7/blob/master/code/cppcon-visitors.cpp) as well as
[pattern matching](https://github.com/solodon4/Mach7/blob/master/code/cppcon-matching.cpp). These are simple enough
to help you get started on your own Mach7 project.

Building sources
----------------

Using GCC (4.4 or later) or Clang (3.3 or later)

    make         - builds .exe files from all the .cpp files in current directory.
    make timings - builds all combinations of encodings, syntax and benchmarks 
                   out of skeleton.cxx for timing purposes
    make syntax  - builds all combinations of configuration flags supported by the 
                   library to make sure nothing was omitted
    make test    - runs all the .exe files in the current folder

Using Visual C++ (2010 or later)

 Mach7 uses its own build.bat script to build all the examples and unit tests that come with it. The script assumes
 each .cpp file to be a standalone program. You can find the most up-to-date list of supported commands by running:

    build.bat /?

 Syntax:

    build [ pgo | tmp | (ver) ] [ filemask*.cpp ... ]
    build [ syntax | timing | cmp | doc | clean | test | check ]

 Commands supported so far:

    build [ pgo | tmp | (ver) ] [ filemask*.cpp ... ] - build given C++ files
    build        - Build all examples using the most recent MS Visual C++ compiler installed
    build syntax - Build all supported library options combination for syntax variations
    build timing - Build all supported library options combination for timing variations
    build cmp    - Build all executables for comparison with other languages
    build doc    - Build Mach7 documentation
    build clean  - Clean all built examples
    build test   - Run all built examples
    build check  - Run those examples for which there are correct_output/*.out files and 
                   check that output is the same

 Modifiers:

    pgo   - Perform Profile-Guided Optimization on produced executables
    tmp   - Keep temporaries
    (ver) - Use a specific version of Visual C++ to compiler the source code. (ver) can be one of the following:
          - 2010 - Visual C++ 10.0
          - 2012 - Visual C++ 11.0
          - 2013 - Visual C++ 12.0

The following batch files do some of these sub-commands directly and have since been integrated into build.bat:
```
 * test-pm-timing.bat - builds all combinations of encodings, syntax and benchmarks out of skeleton.cxx for timing purposes (same as "make timings" for Visual C++)
 * test-pgo.bat - compiles and performs profile-guided optimizations on all files passed as arguments
 * test-pm.bat - builds sources varying amount of derived classes and virtual functions in them
 * test-pm-daily.bat - builds all files in the test suite
 * test-pm-daily-pgo.bat - builds all files in the test suite and performs profile-guided optimizations on them
 * ttt.bat - converts the summary of outputs into a latex definitions used in the performance table
```
Talks
-------------

 * "[Accept No Visitors](http://bit.ly/AcceptNoVisitorsVideo)". [CppCon 2014](http://cppcon.org/). September 12, 2014. Bellevue, WA. [[slides](http://bit.ly/AcceptNoVisitors), [video](http://bit.ly/AcceptNoVisitorsVideo)]
 * "[Mach7: The Design and Evolution of a Pattern Matching Library for C++](http://bit.ly/Mach7CppNowVideo)". [C++ Now 2014](http://cppnow.org). May 14, 2014. Aspen, CO. [[slides](http://bit.ly/Mach7CppNow), [video](http://bit.ly/Mach7CppNowVideo)]

Publications
------------

 * Y.Solodkyy, G.Dos Reis, B.Stroustrup. "[Open Pattern Matching for C++: Extended Abstract](https://parasol.tamu.edu/~yuriys/papers/OPM13EA.pdf)" In Proceedings of the 2013 companion publication for conference on Systems, programming, & applications: software for humanity (SPLASH '13). ACM, New York, NY, USA, pp. 97-98. [pdf](https://parasol.tamu.edu/~yuriys/papers/OPM13EA.pdf), [slides](https://parasol.tamu.edu/~yuriys/presentations/2013-10-27-GPCE-PatternMatching.pptx), [notes](https://parasol.tamu.edu/~yuriys/presentations/2013-10-27-GPCE-PatternMatching-Notes.pdf), [poster](https://parasol.tamu.edu/~yuriys/posters/Mach7.pdf), [project](http://parasol.tamu.edu/mach7/)
 * Y.Solodkyy, G.Dos Reis, B.Stroustrup. "[Open Pattern Matching for C++](https://parasol.tamu.edu/~yuriys/papers/OPM13.pdf)" In Proceedings of the 12th international conference on Generative programming: concepts & experiences (GPCE '13). ACM, New York, NY, USA, pp. 33-42. [pdf](https://parasol.tamu.edu/~yuriys/papers/OPM13.pdf), [slides](https://parasol.tamu.edu/~yuriys/presentations/2013-10-27-GPCE-PatternMatching.pptx), [notes](https://parasol.tamu.edu/~yuriys/presentations/2013-10-27-GPCE-PatternMatching-Notes.pdf), [poster](https://parasol.tamu.edu/~yuriys/posters/Mach7.pdf), [project](http://parasol.tamu.edu/mach7/)
 * Y.Solodkyy. "[Simplifying the Analysis of C++ Programs](https://parasol.tamu.edu/~yuriys/papers/SOLODKYY-DISSERTATION-2013.pdf)" Ph.D. Thesis. Texas A&M University. August 2013. [slides](https://parasol.tamu.edu/~yuriys/presentations/2013-05-22-PhdDefense.pptx)
 * Y.Solodkyy, G.Dos Reis, B.Stroustrup. "[Open and Efficient Type Switch for C++](https://parasol.tamu.edu/~yuriys/papers/TS12.pdf)" In Proceedings of the ACM international conference on Object Oriented Programming Systems Languages and Applications (OOPSLA '12). ACM, New York, NY, USA, pp. 963-982. [pdf](https://parasol.tamu.edu/~yuriys/papers/TS12.pdf), [slides](https://parasol.tamu.edu/~yuriys/presentations/2012-10-25-OOPSLA-TypeSwitch.pptx), [notes](https://parasol.tamu.edu/~yuriys/presentations/2012-10-25-OOPSLA-TypeSwitch-Notes.pdf), [poster](https://parasol.tamu.edu/~yuriys/posters/Mach7.pdf), [extras](https://parasol.tamu.edu/~yuriys/pm/), [project](http://parasol.tamu.edu/mach7/)]

License
-------

Mach7 is licensed under the [BSD License](LICENSE).

Support
-------

Please contact Yuriy Solodkyy at yuriy.solodkyy@gmail.com with any questions regarding Mach7.

Known bugs and limitations
--------------------------

The library is not yet suitable for multi-threaded environment. Lock-free version of vtbl-map is in the works.

The following files crash GCC 4.4.5 on my Fedora 13 box:
    extractor.cpp, shape2.cpp, shape4.cpp, shape5.cpp, shape6.cpp, shape.cpp, numbers.cpp, category.cpp, exp.cpp
If they do on yours too, just delete them, they are all test cases anyways.

For the most up-to-date list of known issues see [Mach7 Issues](https://github.com/solodon4/Mach7/issues).

hi there ~:

for many times, i just write the python code for my self, but today i
think of that may be some day i would want to share my code with other people.
So, the most important thing is that my code needs to be standard formed and
simple, easy-to-read-and-understand.

Thus i talk about the formed of package and the import of modules. here are two
very good blog.

[**habnabblog**](http://blog.habnab.it/blog/2013/07/21/python-packages-and-you/) and
[**Jean-Paul Calderone**](http://as.ynchrono.us/2007/12/filesystem-structure-of-python-project_21.html)


import mode
------------
* implicit relative imports  // BAD practice

* explicit relative imports // just OK

* fully-qualified name of the module aka. absolute imports  // BEAST practice

    advantages:

    1. it makes your code very explicit in what you are importing.

    2. your code will continue to work in the future.

    3. you always know exactly what you’re getting.


where to look up from
----------------------
using absolute imports means that it doesn’t matter where __in the project__ a module is /  when you try to import another module.

**python will always look up the module from the root of you project**


change source directory to a package
------------------------------------
add a `__init__.py` file to each of the subdirectory makes the source directory from a plain directory to a python package

if you’re feeling nice, add README and setup.py to explain and install you package respectively

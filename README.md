DidYouMean-Python
=================

[![Gitter](https://badges.gitter.im/Join Chat.svg)](https://gitter.im/SylvainDe/DidYouMean-Python?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Build Status](https://travis-ci.org/SylvainDe/DidYouMean-Python.svg)](https://travis-ci.org/SylvainDe/DidYouMean-Python)

[![Coverage Status](https://coveralls.io/repos/SylvainDe/DidYouMean-Python/badge.svg?branch=master)](https://coveralls.io/r/SylvainDe/DidYouMean-Python?branch=master)
[![codecov.io](http://codecov.io/github/SylvainDe/DidYouMean-Python/coverage.svg?branch=master)](http://codecov.io/github/SylvainDe/DidYouMean-Python?branch=master)

[![Code Health](https://landscape.io/github/SylvainDe/DidYouMean-Python/master/landscape.svg?style=flat)](https://landscape.io/github/SylvainDe/DidYouMean-Python/master)
[![Code Climate](https://codeclimate.com/github/SylvainDe/DidYouMean-Python/badges/gpa.svg)](https://codeclimate.com/github/SylvainDe/DidYouMean-Python)
[![Code Issues](http://www.quantifiedcode.com/api/v1/project/34c9b3f27db24a1ba944fcf3d69a9d2a/badge.svg)](http://www.quantifiedcode.com/app/project/34c9b3f27db24a1ba944fcf3d69a9d2a)
[![Scrutinizer](https://scrutinizer-ci.com/g/SylvainDe/DidYouMean-Python/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/SylvainDe/DidYouMean-Python/?branch=master)
[![Codacy Badge](https://www.codacy.com/project/badge/54bed0a6466b48ea973325cff2594376)](https://www.codacy.com/app/sylvain-desodt-github/DidYouMean-Python)


Logic to have various kind of suggestions in case of errors (NameError, AttributeError, ImportError, TypeError, ValueError, SyntaxError, MemoryError, OverflowError, IOError, OSError).

Inspired by "Did you mean" for Ruby ([Explanation](http://www.yukinishijima.net/2014/10/21/did-you-mean-experience-in-ruby.html), [Github Page](https://github.com/yuki24/did_you_mean)), this is a simple implementation for/in Python. I wanted to see if I could mess around and create something similar in Python and it seems to be possible.

The logic adding suggestions can be invoked in different ways :

 - a hook on `sys.excepthook`

 - a context manager

 - a post-mortem function for interactive session

 - a function decorator.


See also :

 - [PEP 473 :  Adding structured data to built-in exceptions](http://legacy.python.org/dev/peps/pep-0473/).

 - [dutc/didyoumean](https://github.com/dutc/didyoumean) : a quite similar project developed in pretty much the same time. A few differences though : written in C, works only for AttributeError, etc.

 - [Did You Mean in Perl](http://perltricks.com/article/122/2014/10/31/Implementing-Did-You-Mean-in-Perl)

 - [TheF*ck](https://github.com/nvbn/thefuck) : Correct and execute your previous shell command.

 - [PyDidYouMean](https://github.com/asweigart/pydidyoumean) : Improve "file/command not found" errors with suggestions.


Example
-------

_More examples can be found from the test file `didyoumean/didyoumean_sugg_tests.py`._


### NameError

##### Fuzzy matches on existing names (local, builtin, keywords, modules, etc)

```python
def my_func(foo, bar):
	return foob
my_func(1, 2)
#>>> Before: NameError("global name 'foob' is not defined",)
#>>> After: NameError("global name 'foob' is not defined. Did you mean 'foo' (local)?",)
```
```python
leng([0])
#>>> Before: NameError("name 'leng' is not defined",)
#>>> After: NameError("name 'leng' is not defined. Did you mean 'len' (builtin)?",)
```
```python
import math
maths.pi
#>>> Before: NameError("name 'maths' is not defined",)
#>>> After: NameError("name 'maths' is not defined. Did you mean 'math' (local)?",)
```
```python
passs
#>>> Before: NameError("name 'passs' is not defined",)
#>>> After: NameError("name 'passs' is not defined. Did you mean 'pass' (keyword)?",)
```
```python
def my_func():
	foo = 1
	foob +=1
my_func()
#>>> Before: UnboundLocalError("local variable 'foob' referenced before assignment",)
#>>> After: UnboundLocalError("local variable 'foob' referenced before assignment. Did you mean 'foo' (local)?",)
```
##### Checking if name is the attribute of a defined object

```python
class Duck():
	def __init__(self):
		quack()
	def quack(self):
		pass
d = Duck()
#>>> Before: NameError("global name 'quack' is not defined",)
#>>> After: NameError("global name 'quack' is not defined. Did you mean 'self.quack'?",)
```
```python
import math
pi
#>>> Before: NameError("name 'pi' is not defined",)
#>>> After: NameError("name 'pi' is not defined. Did you mean 'math.pi'?",)
```
##### Looking for missing imports

```python
string.ascii_lowercase
#>>> Before: NameError("name 'string' is not defined",)
#>>> After: NameError("name 'string' is not defined. Did you mean to import string first?",)
```
##### Looking in missing imports

```python
choice
#>>> Before: NameError("name 'choice' is not defined",)
#>>> After: NameError("name 'choice' is not defined. Did you mean 'choice' from random (not imported)?",)
```
##### Special cases

```python
assert j ** 2 == -1
#>>> Before: NameError("name 'j' is not defined",)
#>>> After: NameError("name 'j' is not defined. Did you mean '1j' (imaginary unit)?",)
```
### AttributeError

##### Fuzzy matches on existing attributes

```python
lst = [1, 2, 3]
lst.appendh(4)
#>>> Before: AttributeError("'list' object has no attribute 'appendh'",)
#>>> After: AttributeError("'list' object has no attribute 'appendh'. Did you mean 'append'?",)
```
```python
import math
math.pie
#>>> Before: AttributeError("'module' object has no attribute 'pie'",)
#>>> After: AttributeError("'module' object has no attribute 'pie'. Did you mean 'pi'?",)
```
##### Detection of mis-used builtins

```python
lst = [1, 2, 3]
lst.max()
#>>> Before: AttributeError("'list' object has no attribute 'max'",)
#>>> After: AttributeError("'list' object has no attribute 'max'. Did you mean 'max(list)'?",)
```
##### Trying to find method with similar meaning (hardcoded)

```python
lst = [1, 2, 3]
lst.add(4)
#>>> Before: AttributeError("'list' object has no attribute 'add'",)
#>>> After: AttributeError("'list' object has no attribute 'add'. Did you mean 'append'?",)
```
### ImportError

##### Fuzzy matches on existing modules

```python
from maths import pi
#>>> Before: ImportError('No module named maths',)
#>>> After: ImportError("No module named maths. Did you mean 'math'?",)
```
##### Fuzzy matches on elements of the module

```python
from math import pie
#>>> Before: ImportError('cannot import name pie',)
#>>> After: ImportError("cannot import name pie. Did you mean 'pi'?",)
```
##### Looking for import from wrong module

```python
from itertools import pi
#>>> Before: ImportError('cannot import name pi',)
#>>> After: ImportError("cannot import name pi. Did you mean 'from math import pi'?",)
```
### TypeError

##### Fuzzy matches on keyword arguments

```python
def my_func(abcde):
	pass
my_func(abcdf=1)
#>>> Before: TypeError("my_func() got an unexpected keyword argument 'abcdf'",)
#>>> After: TypeError("my_func() got an unexpected keyword argument 'abcdf'. Did you mean 'abcde'?",)
```
##### Confusion between brackets and parenthesis

```python
lst = [1, 2, 3]
lst(0)
#>>> Before: TypeError("'list' object is not callable",)
#>>> After: TypeError("'list' object is not callable. Did you mean 'list[value]'?",)
```
```python
def my_func(a):
    pass
my_func[1]
#>>> Before: TypeError("'function' object has no attribute '__getitem__'",)
#>>> After: TypeError("'function' object has no attribute '__getitem__'. Did you mean 'function(value)'?",)
```
### ValueError

##### Special cases

```python
'Foo{}'.format('bar')
#>>> Before: ValueError('zero length field name in format',)
#>>> After: ValueError('zero length field name in format. Did you mean {0}?',)
```
### SyntaxError

##### Fuzzy matches when importing from __future__

```python
from __future__ import divisio
#>>> Before: SyntaxError('future feature divisio is not defined',)
#>>> After: SyntaxError("future feature divisio is not defined. Did you mean 'division'?",)
```
##### Various

```python
return
#>>> Before: SyntaxError("'return' outside function", ('<string>', 1, 0, None))
#>>> After: SyntaxError("'return' outside function. Did you mean to indent it, 'sys.exit([arg])'?", ('<string>', 1, 0, None))
```
### MemoryError

##### Search for a memory-efficient equivalent

```python
range(99999999999)
#>>> Before: MemoryError()
#>>> After: MemoryError(". Did you mean 'xrange'?",)
```
### OverflowError

##### Search for a memory-efficient equivalent

```python
range(999999999999999)
#>>> Before: OverflowError('range() result has too many items',)
#>>> After: OverflowError("range() result has too many items. Did you mean 'xrange'?",)
```
### OSError/IOError

##### Suggestion for tilde/variable expansions

```python
os.listdir('~')
#>>> Before: OSError(2, 'No such file or directory')
#>>> After: OSError(2, "No such file or directory. Did you mean '/home/user' (calling os.path.expanduser)?")
```


Usage
-----

I haven't done anything fancy for the installation (yet). You'll have to clone this.

Once you have the code, it can be used in different ways :

 * hook on `sys.excepthook` : just `import didyoumean_hook` and you'll have the suggestions for any exception happening.

 * decorator : just `import didyoumean_decorator` and add the `@didyoumean` decorator before any function (the `main()` could be a good choice) and you'll have the suggestions for any exception happening through a call to that method.

 * I should complete/fix this once it is actually installable because names will most probably change anyway.

Implementation
--------------

All external APIs (decorator, hook, etc) use the same logic behind the scene. It works in a pretty simple way : when an exception happens, we try to get the relevant information out of the error message and of the backtrace to find the most relevant suggestions. To filter the best suggestions out of everything in case of fuzzy match, I am currently using ```difflib```.


Contributing
------------

Feedback is welcome, feel free to :
 * send me an email for any question/advice/comment/criticism
 * open issues if something goes wrong (please provide at least the version of Python you are using).

Also, pull-requests are welcome to :
 * fix issues
 * enhance the documentation
 * improve the code
 * bring awesomeness

As for the technical details :

 * this is under MIT License : you can do anything you want as long as you provide attribution back to this project.
 * I try to follow [PEP 8](http://legacy.python.org/dev/peps/pep-0008/) and [PEP 257](https://www.python.org/dev/peps/pep-0257/) as much as possible. Compliancy is checked during continuous integration using the [pep8](https://pypi.python.org/pypi/pep8) and [pep257](https://pypi.python.org/pypi/pep257) checkers.
 * I try to have most of the code covered by unit tests.
 * I try to write the code in such a way that it works on all Python versions from 2.6 (included).

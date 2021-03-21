# Optimizing with Cython Introduction - Cython Tutorial
Welcome to a Cython tutorial. The purpose of Cython is to act as an intermediary between Python and C/C++. At its heart, Cython is a superset of the Python language, which allows you to add typing information and class attributes that can then be translated to C code and to C-Extensions for Python.

If you've done much Python programming and shared it with your non-Python programmer friends and colleagues (đồng nghiệp) , chances (cơ hội) are, you've been inquired (hỏi thăm) about why you're using Python, since, of course, it's such a "slow" language!

Isn't Python slow? What about the GIL? That dynamic typing though?

I find myself frequently (thường xuyên) defending (bảo vệ) Python by explaining that, while pure Python is indeed (thật) quite slow, Python in practice is not. Libraries like Numpy, Pandas, and Scikit-learn all are C Optimized. When you use them, you're actually making use of C/C++ power, you're just able to use Python syntax. In fact, Numpy, Pandas, and Scikit-learn all make use of Cython! Chances are, the Python+C-optimized code in these popular libraries and/or using Cython is going to be far faster than the C code you might write yourself, and that's if you manage to write it without any bugs.

Despite (bất chấp) there being already many C-Optimized libraries, some times there isn't already a C wrapper for what you're trying to do, and many libraries, or parts of them, are meant to be open enough to not force (buộc) static typing on you, so then you're stuck (bị mắc kẹt) with non optimized code that is slower than C. So, here, we're going to talk about optimizing code with Cython.

At a quick glance (nhìn lướt qua), Cython initially appeared (đã xuất hiện) to me to be quite complex (phức tạp) and imposing (áp đặt), unlikely (k chắc) to be worth (đáng giá) the effort (cố gắng) to learn it. That was until I was sitting in on a Pycon talk about it, and realized (nhận ra) it's actually unbelievably simple, or at least can be. The main crux (mấu chốt): Add typing information...and seriously (nghiêm túc) that's all you need to do to get massive (lớn) gains (lợi nhuận).

What's typing information? In Python, when you declare (khai báo) a variable, like:
```C
x = 5.0
```
You never had to tell the language that the variable 'x' was an integer. In fact, later, you can assign 'Gary' to x and be just fine. This is because Python checks every single time for you to figure out (tìm ra) the type. This is called "dynamic typing."

This is nifty (tiện lợi), and makes learning initially very simple, and Python was really only meant to be a teaching language, but this severely (nghiêm trọng) slows things down.

Instead (thay thế), if we're willing (sẵn lòng) to, we can use static typing and Cython to get some serious (nghiêm trọng) speed ups. Many languages do something more like:
```C
float x = 5.0
```
Cython wants something like:
```C
cdef float x = 5.0
```
Of course you need to keep x as a float, but, as long as you adhere (tuân thủ) to static typing alone, you will be significantly (đáng kể) rewarded (được thưởng). To do this, we need two things:
1. Cython
2. A C/C++ compiler  
For #1, you just simply do pip install cython For #2, things can get a little more hairy depending on your operating system:

Linux: Congratulations, you're probably (có lẽ) done, likely already having a compiler (trình biên dịch). If not, a sudo apt-get install build-essential is likely all you need to do.

Once you have Cython and a compiler, let's go through the Cython workflow and make our own C-Exension! Let's start with a simple python file:
```python
#example_original.py
def test(x):
    y = 0
    for i in range(x):
        y += i
    return y
```
How do we prepare this file to be passed through Cython? Simple, rather than .py, we do .pyx
```python
#example_cython.pyx
def test(x):
    y = 0
    for i in range(x):
        y += i
    return y
```
We obviously don't have any typing information yet. We'll add that in later, but, for now, we'll stick (gắn bó) with this.

Once you have a .pyx, you're ready to build. To do this, we're going to make a setup.py file:
```python
from distutils.core import setup
from Cython.Build import cythonize

setup(ext_modules = cythonize('example_cython.pyx'))
```
Next, in your terminal, do:  
`python setup.py build_ext --inplace`

This should create a build directory, a C file (.c), and a Shared Object file (.so). With this, we can import our C-extension. To illustrate (minh họa) this, you can now delete, or otherwise (nếu k thì) move your example.py and example.pyx files so all that remains (còn lại) is the build, .c and .so files. Now create a new file called testing.py, and we can import our new c extension:
```python
#testing.py
import example_cython

example_cython.test(5)
```
Congratulations! You did it!

So we've not really done any Cython typing...etc, so this code isn't more optimized, but this is actually fairly (công bằng) interesting to show, because it was very simple to do, and it illustrates that you can do as much, or as little, as you want to with Cython implementation (triển khai).

Now, we'll begin adding typing information. Let's go over some of the typing declarations (khai báo):  
**cdef declarations:¶**  
```C
cdef int x,y,z
cdef char *s
cdef float x = 5.2 (single precision)
cdef double x = 40.5 (double precision)
cdef list languages
cdef dict abc_dict
cdef object thing
```  
**def, cdef, and cpdef¶**  
def - regular (bình thường) python function, calls from Python only.  
cdef - cython only functions, can't access these from python-only code, must access within Cython, since there will be no C translation to Python for these.  
cpdef - C and Python. Will create a C function and a wrapper for Python. Why not *always* use cpdef? In some cases, you might have C only pointer, like a C array. We'll be mostly using cpdef, however.

Now, we're going to start with the same code from before:
```python
#example_original.py
def test(x):
    y = 0
    for i in range(x):
        y += i
    return y
```
Now let's save this file as example_cython.pyx, and begin to make some changes.
```python
#example_cython.pyx
def test(int x):
    cdef int y = 0
    cdef int i
    for i in range(x):
        y += i
    return y
```
Above, we've given the input parameter, and the two values that we will be using some typing information. Let's build and test this now.
```python
#setup.py
from distutils.core import setup
from Cython.Build import cythonize

setup(ext_modules = cythonize('example_cy.pyx'))
```
Now we can test the outcome with:
```python
#testing_things.py
import timeit

cy = timeit.timeit('''example_cy.test(5)''',setup='import example_cy',number=100)
py = timeit.timeit('''example.test(5)''',setup='import example', number=100)

print(cy, py)
print('Cython is {}x faster'.format(py/cy))
```
Running this in terminal:  
`python3 testing_things.py
5.769999916083179e-06 4.714800024885335e-05
Cython is 8.171230664567945x faster`  
Not bad! There are a few other minor (nhỏ) changes we could make, like:
```C
cpdef int test(int x):
    cdef int y = 0
    cdef int i
    for i in range(x):
        y += i
    return y
 ```
But this isn't necessarily (nhất thiết) all that much better (run it a few times, or use a higher iteration (sự lặp lại) count with timeit).

So where are our gains coming from? Well, all we're doing is adding typing information, so where would that increase our speeds the most? Anywhere where we're using variables the most frequently. In our case, that'd be the for loop. So, if we wanted to impress (gây ấn tượng) our friends of our Cython powers, and how much more speed we can get out of Cython, all we need to do is make x larger. So this time:
```C
#testing_things.py
import timeit

cy = timeit.timeit('''example_cy.test(5000)''',setup='import example_cy',number=100)
py = timeit.timeit('''example.test(5000)''',setup='import example', number=100)

print(cy, py)
print('Cython is {}x faster'.format(py/cy))
```
Again in terminal:  
`$ python3 testing_things.py
0.0002787369999168732 0.04767731600031766
Cython is 171.04767581819533x faster`  
We're wizards (thuật sĩ)!

One option we have to analyze areas where we could attempt (cố gắng) to use cython is via (thông qua) cythonize's html output. For example, let's take our original Python script, convert to .pyx:
```python
#furthertesting.pyx
def test(x):
    y = 0
    for i in range(x):
        y += i
        return y
```
Now, in your terminal, you can do:  

`$ cython -a furthertesting.pyx`  

This should give you a furthertesting.html (thử nghiệm thêm) file:

This will produce a C file along with a .html file. Open that .html file, and you can see lines highlighted in yellow in accordance to their approximate proximity to Python. This isn't perfect, and you wont always be able to improve things based on this output, but it can become helpful to you to locate areas that you could possibly improve. For example, we can generate this same HTML file for our actual cython file:

`$ cython -a example_cy.pyx`
See more at [here](https://pythonprogramming.net/introduction-and-basics-cython-tutorial/)

Now we can see that the only relation to Python is our cpdef, since we wanted to be able to use this function in Python.

Okay, that's all for now for Cython. I may bring in more advanced topics in the future, but, believe it or not, most of your gains will come purely from using static typing. You can also look into various commands like "with nogil." Cython can get quite a bit more complicated for you if you're up for it. If you're familiar with C/C++, I highly recommend you dive in more. Otherwise, consider places in your code where Python has to keep verifying the type of some variable. This can either be in loops, or in programs that scale out. For example, if you have a heavily trafficked website, or maybe you've got some sort of crawlbot, or maybe you're analyzing tick prices from stocks, any time you're scaling out the use of variables, you should consider adding typing information for some serious performance improvements.

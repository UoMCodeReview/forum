Reviewing code can be intimidating, especially if you don't consider yourself as an experienced programmer.
What if you don't find anything to say?

It is a common misconception to think that reviewers must necesseraly be advanced developpers, and that they must find bugs or make tons of comments for the review to be successful.

Instead, your main job as a reviewer is **simply to ask questions**. This should be easy, as this is rather unlikely that you will understand every line of code without clarifications!
You first goal as a reviewer is to force the code author to explain the detail of their code to you. Doing so, they will very probalby identify bugs and area of improvements **themselves**.

Once you're confortable reading somebeody else's code and asking questions, the next step is to look for potential issues yourself.
If you have no idea what to look for, here is a list of the major points to check in a code review.

## What to look for as a reviewer

1. [Naming](#naming)
2. [Duplicated code](#duplicated)
3. [Long functions](#long)
4. [Complex if statements](#complexity)
5. [Obscure lines](#obscure)
4. [Unintended behavior](#unintended)
5. [Comments](#comments)
6. [Performance low hanging fruits](#performance)
7. [Use of built-in functions](#builtin)
8. [Leveraging third-party libraries](#third-party)

*Under construction*

The following points are common guidelines, not rules. Cases may arise where it is best not to follow them.

### Code style

Whatever the programming language, there is certainly a style guide to follow:

| Language   | Style guide                  | Tool                      |
|:----------:|:----------------------------:|:-------------------------:|
| Python     | PEP8                         | pycodestyle               |
| C++        | C++ Core Guidelines          | clang-tidy                |
| R          | Hadley Wickham's style guide | lintr                     |
| JavaScript | JavaScript Standard style    | JavaScript Standard style |
| Fortran    | Fortran Best Practices       | gfortran                  |
| Ruby       | Ruby Style Guide             | RuboCop                   |

Following a style guide makes sure that your code is written in a way that is consistent with code written
by other programmers (assuming they also follow the style guide).

- You code will be easier to read/understand for programmers outside your project.
- Code style will be consistent throughout the project even if several developpers are working on it.
- Style guides are based on best practices for the language.

Style guides are well worth the read, but often lenghty and sometimes obscure. Luckiliy, there exist many
software tools to enforce style guides automatically (see table above).

<a id="naming"></a>

### Naming
Knowingly the hardest part in software developement.

- Always use descriptive names at every levels, whether it is for variables, functions/methods or classes. Name of functions and classes should convey intent.

> when implmenting mathematical expresions, it's often tempting to name the variable after its mathematical symbol (e.g. `alpha`, `m`, `R0`..).
> This is not recommended, as this makes the code less readable, and other people may use different notations. Use explicit names instead
> (e.g. `streamwise_velocity_field`, `current`, `infection_rate`...)

- Avoid "magic numbers"
```C
for (i=0; i<26;i++){
```
should be
```C
int AlphabetSize = 26;
for (i=0; i<AlphabetSize ;i++){
```
[See this post by Chris Bertrand](https://dev.to/designpuddle/code-review-checklist-14ke).

<a id="duplicated"></a>

### Duplicated code
Copy pasting code may speed up development in the short term... but
- It cripples the code's maitainability and extendability (either by a colleague or yourself three months down the line).
- Each time you modify a part, you have to remember to modify all duplicated parts without forgetting any. Not only it is boring work, but also error prone.
- Duplicated code also decreases readability, as your code is unnecesraly longer, and makes bug hunting much harder.

Typical solutions include
- Defining new functions/methods that can be reused in different parts of the code
- Use class inheritance or composition

<a id="complex"></a>

### Complex `if` statements

Complex `if` statements make your code much less readable, as it forces the reader to hold and process a lot
of information simultaneously.

The following `if` statement determines if yes or no a point `(x,y)` is contained inside a rectangle:
```python
	if (x > xmin and x < xmax and y > ymin and y < ymax):
```

The above complex condition can be replaced by a function call:
```python
def is_inside_rectangle(x,y):
    x_in = x > xmin and x < xmax
    y_in = y > ymin and y < ymax

    return x_in and y_in

# ...
if is_inside_rectangle(x,y):
```

Another common problematic construct is nested `if-else` statements:
```python
nb_of_events = len(events)
if nb_of_events == 1:
    list_of_available_events = []
else:
    if events_subsequent:
        list_of_available_events = [1]
    else:
        list_of_available_events = []
        for i in range(nb_of_events):
            list_of_available_events.append(i)
```

The above can be better written as:
```python
def get_list_of_available_events(events, events_subsequent):
    nb_of_events = len(events)
    if nb_of_events == 1:
        return []
    if events_subsequent:
        return [1]
    return range(nb_of_events)

list_of_available_events = get_list_of_available_events()
```

Complex `if` statements and nested `if-else` significantly hinder readability.
They also make your code much harder to test, as you'll have to write a test for each possible branch
in your code.

See the notions of [cyclomatic complexity](https://docs.codeclimate.com/docs/cyclomatic-complexity) and
[cognitive complexity](https://docs.codeclimate.com/docs/cognitive-complexity).
See also [Writing simpler and more maintainable Python by Anthony Shaw (video)](https://youtu.be/dqdsNoApJ80?t=303)

<a id="long"></a>

### Long functions/methods
- Functions should be as short as possible.
  Readibility an modularity.
- Functions should do one thing.
  Facilitates testing.

<a id="obscure"></a>

### Obscure lines

Modern programming language such as Python, Ruby or even modern C++ provide powerful functionalities allowing
programmers to do more, typing less. Although these can lead to shorter and more descriptive code, it is a double-edged sword.

Consider the following line of python

```python
for ensemble in zip(*[traj_sample(x0, t0, *args, **kwargs) for _ in range(nsamples)]):
```

The above line relies on
- A generator function call
- List comprehension
- List unpacking
- The `zip` built-in function

That's a lot. Although this is nice and short, this is difficult to read.

Similarly to mathematical proofs, doing too much in one step makes the argument harder to follow.

In the above, simply adding an extra line allow to use a descriptive intermediate variable:

```python
list_of_generators = [traj_sample(x0, t0, *args, **kwargs) for _ in range(nsamples)]
for ensemble in zip(*list_of_generators):

```

Resist the clever one-liners !

<a id="unintended"></a>

### Unintended behavior
A typical example is a function with parameters that are constrained (e.g. strictly positive, integer value...)
.
The following C function is compiled without errors:
```C
double returnArrayElement(int i, double *array){

  return array[i];

}
```
However, if `i` is negative, or larger than the total allocated size of `array`, executing the code will result in a Segmentation Fault.

> Code should not trust its user, whether the user is a human or some other code.

The corrolary to the above statement is a coding style known as [defensive programming](https://swcarpentry.github.io/python-novice-inflammation/10-defensive/index.html).

Example of a defensive python function:
```python
def compute_acceleration(mass, total_force_on_body):
	if mass <= 0:
		raise ValueError("Mass of body must be strictly positive")
	return total_force_on_body/mass

```

<a id="documented"></a>

### Undocumented functions, classes and modules

Any logical structure (function, class, module) should be accompanied by a documentation string (commonly known as *docstring*).
Example
```python
def compute_acceleration(mass, total_force_on_body):
    """
	Compute and return acceleration on body, according to Newton's 2nd law

	Parameters
	----------
	mass: float
	  Mass of body
	total_force_on_body: float
	  Sum of all forces exerted on the body

	Returns:
	--------
	a: float
      The acceleration
	"""
	if mass <= 0:
		raise ValueError("Mass of body must be strictly positive")
	return total_force_on_body/mass

```

<a id="comments"></a>

### Comments
Commenting can be a confusing topic, since the general advice is
> Comment you code, but not too much

The above guideline can be understood by taking a rather extreme stance:

> Good code does not need comment to be understood.

The rationale is that, most of the time, comments can be avoided by using more descriptive names, shorter methods, and simpler constructs.

Comments shpuld describe the *why*, not the *what*.


<a id="performance"></a>

### Performance low hanging fruits
Typical examples include

- Ordering of nested loops

```fortran
implicit none

   integer:: i, j

   iloop: do i = 1, mesh_size_x
         jloop: do j = 1, mesh_size_y

	        ! Make sure inner index is first
            a(i, j) = prefactor(i, j)*(term1(i, j) + term2(i, j))

      end do jloop
   end do iloop
```

- Use of local variables
```C++
for (int i; i<stop;i++)
{
  local_var = vector[i];
  result = 2.*local_var*local_var + local_var + 4.
  ...

```

<a id="builtin"></a>

### Potential use of built in functions
Do not not reinvent the wheel!

Programming languages usually come with useful libraries that implement common tasks.
```python
from itertools import accumulate

accumulate([1,2,3,4,5], initial=100) --> 100 101 103 106 110 115
```

<a id="third-party"></a>

### Potential use of third party libraries
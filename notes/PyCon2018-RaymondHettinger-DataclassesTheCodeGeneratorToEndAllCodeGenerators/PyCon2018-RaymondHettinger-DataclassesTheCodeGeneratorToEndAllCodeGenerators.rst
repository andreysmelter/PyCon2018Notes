Dataclasses: The code generator to end all code generators
==========================================================

What is a code generator?

1. It is a tool that writes code for based on provided specification.
2. It can save time.
3. It reduces wordiness.  

What are dataclasses for?

1. It makes a mutable data holder, in the spirit of named tuples.
2. It writes boiler-plate code for you, simplifying the process of writing the class.

Dataclass is *roughly* a "mutable named tuple with defaults".


Examples
--------

Dataclass version
~~~~~~~~~~~~~~~~~

.. code:: python

    from dataclasses import dataclass

    @dataclass
    class Color:
        hue: int
        saturation: float
        lightness: float = 0.5

* ``dataclass`` uses a class decorator.


Namedtuple version
~~~~~~~~~~~~~~~~~~

.. code:: python

    from typing import NamedTuple

    class Color(NamedTuple):
        hue: int
        saturation: float
        lightness: float = 0.5

* ``namedtuple`` uses inheritance with a metaclass.


Working with the dataclass
~~~~~~~~~~~~~~~~~~~~~~~~~~

>>> from dataclasses import asdict, astuple, replace
>>> c = Color(33, 1.0)
>>> c
Color(hue=33, saturation=1.0, lightness=0.5)  # repr came for free, it lets you see how it was created

>>> c.hue  # you can access fields by name
33
>>> c.saturation
1.0
>>> c.lightness
0.5

>>> replace(c, hue=120)  # function that allolws you to replace fields
Color(hue=120, saturation=1.0, lightness=0.5)
>>> asdict(c)  # can turn it into a regular dictionary
{'hue': 120, 'saturation': 1.0, 'lightness': 0.5}
>>> astuple(c)  # can turn it into a regular tuple 
(120, 1.0, 0.5)

>>> Color.__annotations__  # it builds type annotations for you
{'hue': <class 'int'>, 'saturation': <class 'float'>, 'lightness': <class 'float'>}

>>> c.hue = 66  # NEW: you can assign a value (dataclasses are mutable)
>>> c
Color(hue=66, saturation=1.0, lightness=0.5)

>>> import sys
>>> sys.getsizeof(c) + sys.getsizeof(vars(c))
168  # bytes

>>> import timeit
>>> min(timeit.repeat('c.hue', 'from __main__ import c'))
0.0333 # nanoseconds


Working with the namedtuple
~~~~~~~~~~~~~~~~~~~~~~~~~~~

>>> from typing import NamedTuple
>>> c = Color(33, 1.0)
>>> c
Color(hue=33, saturation=1.0, lightness=0.5)  # has the same appearance

>>> c.hue  # has the same name field access
33
>>> c.saturation
1.0
>>> c.lightness
0.5

>>> c._replace(hue=120)  # methods start with an underscore. NOTE: _ is not a private method in this case (prevent namespace conflicts with actual field names)!
Color(hue=120, saturation=1.0, lightness=0.5)
>>> c._asdict()  # can turn it into a regular dictionary  # OrderedDict instead of regular dict
OrderedDict([('hue', 120), ('saturation', 1.0), ('lightness', 0.5)])
>>> tuple(c)  # can turn it into a regular tuple 
(120, 1.0, 0.5)

>>> Color.__annotations__  # it builds type annotations for you
OrderedDict([('hue', <class 'int'>), ('saturation', <class 'float'>), ('lightness', <class 'float'>)])

>>> hue, saturation, luminosity = c  # unpackable

>>> import sys
>>> sys.getsizeof(c)
72  # bytes

>>> import timeit
>>> min(timeit.repeat('c.hue', 'from __main__ import c'))
0.0614 # nanoseconds


Summary of differences
----------------------

*Dataclass* | *NamedTuple*

-  *replace()* function | *_replace()* method
-  *asdict()* function | *_asdict()* method
-  converts to regular *dict* | converts to *OrderedDict*
-  *astuple()* function | *tuple()* function
-  mutable | frozen
-  unhashable | hashable
-  non-iterable | iterable and unpackable
-  no comparison methods | sortable
-  underlying store: instance *dict* | underlying store: *tuple*
-  ~168 bytes | ~72 bytes
-  ~33 ns access | ~61 ns access


Generated code
--------------

Code you write
~~~~~~~~~~~~~~

.. code:: python

    from dataclasses import dataclass

    @dataclass
    class Color:
        hue: int
        saturation: float
        lightness: float = 0.5


Code being generated
~~~~~~~~~~~~~~~~~~~~

.. code:: python

    from dataclasses import Field, _MISSING_TYPE, _DataclassParams

    class GeneratedColor:
        'Color(hue: int, saturation: float, lightness: float = 0.5)'
        
        def __init__(self, hue: int, saturation: float, lightness: float = 0.5) -> None:
            self.hue = hue
            self.saturation = saturation
            self.lightness = lightness
            
        def __repr__(self):
            return (self.__class__.__qualname__ +  # Note: higher quality repr!
                    f"(hue={self.hue!r}, saturation={self.saturation!r}, lightness={self.lightness!r})")
      
        def __eq__(self, other):
            if other.__class__ is self.__class__:  # Note: compare aples to aples!
                return (self.hue, self.saturation, self.lightness) == (other.hue, other.saturation, other.lightness)
            return NotImplemented
        
        __hash__ = None  # Note: whenever you write an equality method you also need to say something about hashing!
                         # If you left this out it would compare on identity
        
        hue: int
        saturation: float
        lightness: float = 0.5

        # Class introspection:

        __dataclass_params__ = _DataclassParams(
            init=True,
            repr=True,
            eq=True,
            order=False,
            unsafe_hash=False,
            frozen=False)
        
        __dataclass_fields__ = {
            'hue': Field(default=_MISSING_TYPE,
                         default_factory=_MISSING_TYPE,
                         init=True,
                         repr=True,
                         hash=None,
                         compare=True,
                         metadata={}),
            'saturation': Field(default=_MISSING_TYPE,
                                default_factory=_MISSING_TYPE,
                                init=True,
                                repr=True,
                                hash=None,
                                compare=True,
                                metadata={}),
            'lightness': Field(default=0.5,
                               default_factory=_MISSING_TYPE,
                               init=True,
                               repr=True,
                               hash=None,
                               compare=True,
                               metadata={})
        }
        __dataclass_fields__['hue'].name = 'hue'
        __dataclass_fields__['hue'].type = int
        __dataclass_fields__['saturation'].name = 'saturation'
        __dataclass_fields__['saturation'].type = float
        __dataclass_fields__['lightness'].name = 'lightness'
        __dataclass_fields__['lightness'].type = float


Working with dataclasses: uncommon case
---------------------------------------

Freezing and Ordering
~~~~~~~~~~~~~~~~~~~~~

* For common case, dataclasses are mutable (can not be used as set elements or dictionary keys).
* For common case, dataclasses do not have order.
* When we need this features, we can easily override it in the dataclass dedcorator specification:


Code you write
~~~~~~~~~~~~~~

.. code:: python

    from dataclasses import dataclass

    @dataclass(order=True, frozen=True)  # Note: change specification accoring to your needs!
    class Color:
        hue: int
        saturation: float
        lightness: float = 0.5


Working with the mutable and orderable dataclass
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

>>> colors = [Color(33, 1.0), Color(66, 0.75), Color(99, 0.5), Color(66, 0.75)]

>>> sorted(colors)
[Color(hue=33, saturation=1.0, lightness=0.5), 
 Color(hue=66, saturation=0.75, lightness=0.5),
 Color(hue=66, saturation=0.75, lightness=0.5),
 Color(hue=99, saturation=0.5, lightness=0.5)]

>>> set(colors)
{Color(hue=33, saturation=1.0, lightness=0.5), 
 Color(hue=66, saturation=0.75, lightness=0.5),
 Color(hue=99, saturation=0.5, lightness=0.5)}


Code being generated
~~~~~~~~~~~~~~~~~~~~

.. code:: python

    def __lt__(self, other):  # Note: additional comparison methods are written for you!
        if other.__class__ is self.__class__:
            return (self.hue, self.saturation, self.lightness,) < (other.hue, other.saturation, other.lightness,)
        return NotImplemented

    def __le__(self, other):  # Note: additional comparison methods are written for you!
        if other.__class__ is self.__class__:
            return (self.hue, self.saturation, self.lightness,) <= (other.hue, other.saturation, other.lightness,)
        return NotImplemented

    def __gt__(self, other):  # Note: additional comparison methods are written for you!
        if other.__class__ is self.__class__:
            return (self.hue, self.saturation, self.lightness,) > (other.hue, other.saturation, other.lightness,)
        return NotImplemented

    def __ge__(self, other):  # Note: additional comparison methods are written for you!
        if other.__class__ is self.__class__:
            return (self.hue, self.saturation, self.lightness,) >= (other.hue, other.saturation, other.lightness,)
        return NotImplemented

    def __setattr__(self, name, value):  # Note: method that respect the fact that it is frozen! 
        if type(self) is cls or name in ('hue', 'saturation', 'lightness',):
            raise FrozenInstanceError(f"cannot assign to field {name!r}")
        super(cls, self).__setattr__(name, value)  # Note: class is partially writable!

    def __delattr__(self, name, value):  # Note: method that respect the fact that it is frozen!
        if type(self) is cls or name in ('hue', 'saturation', 'lightness',):
            raise FrozenInstanceError(f"cannot assign to field {name!r}")
        super(cls, self).__delattr__(name, value)  # Note: class is partially deletable!

    def __hash__(self):  # Note: hash is not None!
        return hash((self.hue, self.saturation, self.lightness,))


More customization specifications
---------------------------------

* Field factories: ``field(default_factory) = list``
* Custom methods: adding method to a dataclass is no different than for any other class.
* Limiting hashing to immutable fields: ``field(hash=False)``
* Limiting fields which are displayed: ``field(repr=False)``
* Limiting which fields are used in comparisons: ``field(compare=False)``
* Attaching metadata: ``metadata={'units': 'bitcoin'}``


Customization Example
~~~~~~~~~~~~~~~~~~~~~

.. code:: python

    from verbose_dataclasses import dataclass, field
    from datetime import datetime

    @dataclass(order=True, unsafe_hash=True)
    class Employee:
        emp_id: int = field()
        name: str = field()
        gender: str = field()
        salary: int = field(hash=False, repr=False, metadata={'units': 'bitcoin'})
        age: int = field(hash=False)
        viewed_by: list = field(default_factory=list, compare=False, repr=False)

        def access(self, viewer_id):
            self.viewed_by.append((viewer_id, datetime.now()))


Code being generated
~~~~~~~~~~~~~~~~~~~~

.. code:: python

    def __init__(self, emp_id: int, name: str, salary: int, age: int, viewed_by: list=_HAS_DEFAULT_FACTORY):
        self.emp_id = emp_id
        self.name = name
        self.gender = gender
        self.salary = salary
        self.age = age
        self.viewed_by = list() if viewed_by is _HAS_DEFAULT_FACTORY else viewed_by

    def __repr__(self):
        return (self.__class__.__qualname__ +
                f"(emp_id={self.emp_id!r}, name={self.name!r}, gender={self.gender!r}, age={self.age!r})")

    def __eq__(self, other):
        if other.__class__ is self.__class__:  # Note: compare aples to aples!
            return (self.emp_id, self.name, self.gender, self.salary, self.age,) == (other.emp_id, other.name, other.gender, other.salary, other.age,)
        return NotImplemented

    def __lt__(self, other):
        if other.__class__ is self.__class__:
            return (self.emp_id, self.name, self.gender, self.salary, self.age,) < (other.emp_id, other.name, other.gender, other.salary, other.age,)
        return NotImplemented

    def __le__(self, other):
        if other.__class__ is self.__class__:
            return (self.emp_id, self.name, self.gender, self.salary, self.age,) <= (other.emp_id, other.name, other.gender, other.salary, other.age,)
        return NotImplemented

    def __gt__(self, other):
        if other.__class__ is self.__class__:
            return (self.emp_id, self.name, self.gender, self.salary, self.age,) > (other.emp_id, other.name, other.gender, other.salary, other.age,)
        return NotImplemented

    def __ge__(self, other):
        if other.__class__ is self.__class__:
            return (self.emp_id, self.name, self.gender, self.salary, self.age,) >= (other.emp_id, other.name, other.gender, other.salary, other.age,)
        return NotImplemented

    def __hash__(self):
        return hash((self.emp_id, self.name, self.gender,))


Links
-----

* Talk: https://youtu.be/T-TwcmT6Rcw
* PEP 557: https://www.python.org/dev/peps/pep-0557/
* Raymond Hettinger on Twitter: `@raymondh`_

.. _@raymondh: https://twitter.com/raymondh

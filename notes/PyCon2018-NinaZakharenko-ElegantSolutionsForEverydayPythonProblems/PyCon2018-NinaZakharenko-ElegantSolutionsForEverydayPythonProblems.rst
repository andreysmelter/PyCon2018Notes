PyCon 2018 - Nina Zakharenko - Elegant Solutions For Everyday Python Problems
=============================================================================


What is elegant code?
---------------------

* How do we make code elegant?
    - We pick the right tool for the job!
* Beauty is in the eye of the beholder.


Magic methods
-------------

* Magic methods start and end with a double underscore (dunder).
* By implementing a few straightforward magic methods, you can 
  make your objects behave like built-ins.

.. code:: python

    class Money:

        currency_rates = {
            '$': 1,
            '€': 0.88
        }

        def __init__(self, symbol, amount):
            self.symbol = symbol
            self.amount = amount

        def __str__(self):
            return '{}{:.2f}'.format(self.symbol, self.amount)

        def convert(self, other):
            """Convert other amount to our currency"""
            new_amount = other.amount / self.currency_rates[other.symbol] * self.currency_rates[self.symbol]
            return Money(self.symbol, new_amount)

        def __add__(self, other):
            """ Add 2 Money instances using '+' """
            new_amount = self.amount + self.convert(other).amount
            return Money(self.symbol, new_amount)


.. code:: python

    class SquareShape:
        def __len__(self):
            """ Return the number of sides in our shape """
            return 4


Magic methods examples
~~~~~~~~~~~~~~~~~~~~~~


__str__ in action
*****************

.. code:: python

    >>> soda_cost = Money('$', 5.25)
    >>> print(soda_cost)
    $5.25
    >>> pizza_cost = Money('€', 7.99)
    €7.99

__add__ in action
*****************

.. code:: python

    >>> # __add__ in action
    >>> print(soda_cost + pizza_cost)
    $14.33
    >>> print(pizza_cost + soda_cost)
    €12.61

__getitem__ in action
*********************

.. code:: python

    >>> # __getitem__ maps to brackets
    >>> d = {'one': 1, 'two': 2}
    >>> d['two']
    2
    >>> d.__getitem__('two')
    2

__len__ in action
*****************

.. code:: python

    >>> square = SquareShape()
    >>> len(square)
    4


Custom iterators
----------------

* A ``class`` needs to implement ``__iter__()`` method.
* ``__iter__()`` must return an iterator.
* ``__next__()`` also need to be implemented and ``raise StopIteration`` when there are no more items to return.


Custom iterator examples
~~~~~~~~~~~~~~~~~~~~~~~~

* We have a server running services on different ports.
* We want to loop over only active services.


Custom iterator with __iter__ and __next__
******************************************

.. code:: python

    class IterableServer:

        services = [
            {'active': False, 'protocol': 'ftp', 'port': 21},
            {'active': True, 'protocol': 'ssh', 'port': 22},
            {'active': True, 'protocol': 'http', 'port': 80},
        ]

        def __init__(self):
            self.current_pos = 0

        def __iter__(self):
            # can return self, because __next__ is implemented
            return self

        def __next__(self):
            while self.current_pos < len(self.services):
                service = self.services[self.current_pos]
                self.current_pos += 1
                if service['active']:
                    return service['protocol'], service['port']
            raise StopIteration

.. code:: python

    >>> # loop over all active services
    >>> for protocol, port in IterableServer():
            print('service {} on port {}'.format(protocol, port))
    service ssh on port 22
    service http on port 80
    >>> 

Custom iterator with yield
**************************

* Use a ``generator`` when your iterator **does not** need to maintain a lot of state.

.. code:: python

    class IterableServer:

        services = [
            {'active': False, 'protocol': 'ftp', 'port': 21},
            {'active': True, 'protocol': 'ssh', 'port': 22},
            {'active': True, 'protocol': 'http', 'port': 80},
        ]

        def __iter__(self):
            for service in self.services:
                if service['active']:
                    yield service['protocol'], service['port']


.. code:: python

    >>> # loop over all active services
    >>> for protocol, port in IterableServer():
            print('service {} on port {}'.format(protocol, port))
    service ssh on port 22
    service http on port 80
    >>> 


Method magic
------------

Alias methods
~~~~~~~~~~~~~

.. code:: python

    class Word:

        def __init__(self, word):
            self.word = word

        def __repr__(self):
            return self.word

        def __add__(self, other_word):
            return Word('{} {}'.format(self.word, self.other_word))

        # Add an alias
        concat = __add__

* We can add an alias from __add__ to concat because methods are just objects.

.. code:: python

    >>> # remember, concat = __add__
    >>> first_name = Word('Max')
    >>> last_name = Word('Smith')
    >>>
    >>> first_name + last_name
    Max Smith
    >>>
    >>> first_name.concat(last_name)
    Max Smith
    >>>
    >>> Word.__add__ == Word.concat
    True
    >>>


getattr(object, name, default)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


.. code:: python

    >>> class Dog:
            sound = 'Bark'
            def speak(self):
                print(self.sound + '!', self.sound + '!')
    >>> 
    >>> my_dog = Dog()
    >>> my_dog.speak()
    Bark! Bark!
    >>> 
    >>> getattr(my_dog, 'speak')
    <bound method Dog.speak of <__main__.Dog object at 0x7f6d8c3c97b8>>
    >>> 
    >>> speak_method = getattr(my_dog, 'speak')
    >>> speak_method()
    Bark! Bark!
    >>> 


Example: CLI tool with dynamic commands
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: python

    class Operations:
        
        def say_hi(self, name):
            print('Hello,', name)
    
        def say_bye(self, name):
            print('Goodbye,', name)
    
        def default(self, arg):
            print('This operation is not supported.')
    
    if __name__ == '__main__':
        operations = Operations()
        # let's assume error handling
        command, argument = input('> ').split()
        getattr(operations, command, operations.default)(argument)


.. code:: bash

    $ python demo.py

    > say_hi Nina
    Hello, Nina
    
    > blah blah
    This operation is not supported.



functools.partial(func, *args, **kwargs)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Return a new **partial object** which behaves like ``func``
  called with ``args`` & ``kwargs``.

* If more arguments are passed in, they are appended to ``args``.

* If more keyword arguments are passed in, they extend and override ``kwargs``.


.. code:: python

    >>> # We want to be able to call this function on any int
    >>> # without having to specify the base
    >>> int('10010', base=2)
    18 

    >>> from functools import partial
    >>> basetwo = partial(int, base=2)
    >>> basetwo
    <functools.partial object at 0x1085a09f0>
    >>>
    >>> basetwo('10010')
    18


Context managers
----------------

When should I use one?
~~~~~~~~~~~~~~~~~~~~~~

* Need to perform an action before and/or after an operation.

    * Closing a resource after you are done with it (file, network connection).
    * Perform cleanup before/after a function call.

* Turn features of your application on and off easily.

    * A/B testing.
    * Rolling releases.
    * Show beta version to users opted-in beta testing program.

.. code:: python

    class FeatureFlags:

        SHOW_BETA = 'Show Beta version of Home Page'

        flags = {
            SHOW_BETA = True
        }

        @classmethod
        def is_on(cls, name):
            return cls.flags[name]

        @classmethod
        def toggle(cls, name, value):
            cls.flags[name] = value

    feature_flags = FeatureFlags()


* How do we temporarily turn features on and off when testing flags?


.. code:: python

    # example code that we would like to write
    with feature_flag(FeatureFlags.SHOW_BETA):
        assert '/beta' == get_homepage_url()


.. code:: python

    class feature_flag:
        
        """Implementing a Context Manager"""

        def __init__(self, name, on=True):
            self.name = name
            self.on = on
            self.old_value = feature_flags.is_on(name)

        def __enter__(self):
            feature_flags.toggle(self.name, self.on)

        def __exit__(self, *args):
            feature_flags.toggle(self.name, self.old_value)


* The better way: using the contectmanager decorator

.. code:: python

    from contextlib import contextmanager

    @contextmanager
    def feature_flag(name, on=True):
        old_value = feature_flags.is_on(name)
        feature_flags.toggle(name, on)         # behavior of __enter__()
        yield                                  # give up control
        feature_flags.toggle(name, old_value)  # clean up: behavior of __exit__()

.. code:: python

    def get_homepage_url():
        """Returns the path of the page to display."""
        if feature_flags.is_on(FeatureFlags.SHOW_BETA):
            return '/beta'
        else:
            return '/homepage'


    def test_homepage_url_with_context_manager():

        with feature_flag(FeatureFlags.SHOW_BETA):
            assert get_homepage_url() == '/beta'
            print('Seeing the beta homepage...')

        with feature_flag(FeatureFlags.SHOW_BETA, on=False):
            assert get_homepage_url() == '/homepage'
            print('Seeing the standard homepage...')


Decorators
----------

* The simple explanation:
   - Syntactic sugar that allows modification of an underlying function.

* Wrap a function in another function.
* Do something:
   - Before the call.
   - After the call.
   - With provided arguments.
   - Modify the return value or arguments.


.. code:: python

    class User:

        is_authenticated = False

        def __init__(self, name):
            self.name = name


    def display_profile_page(user):
        """Display profile page for logged in User."""

        # thrown an exception if trying to access data only for logged in users 
        if not user.is_authenticated:
            raise Exception('User must login.')

        print('Profile: {}'.format(user.name))

* We can use a decorator to remove some code duplication.

.. code:: python

    def enforce_authentication(func):
        def wrapper(user):
            if not user.is_authenticated:
                raise Exception('User must login.')
            return func(user)
        return wrapper

* Using ``enforce_authentication`` without a decorator:

.. code:: python

    enforce_authentication(display_profile_page)(some_user)

* Using ``enforce_authentication`` with a decorator:

.. code:: python

    @enforce_authentication
    def display_profile_page(user):
        print('Profile: {}'.format(user.name))

* Common uses for decorators:
   - logging
   - timing
   - validation
   - rate limiting
   - mocking/patching 


ContextDecorators
-----------------

* ContextManager + Decorator combined.
* By using ``ContextDecorator`` you can easily write classes that can be 
  used both as decorators with ``@`` **and** context managers with 
  the ``with`` statement.
* ``ContextDecorator`` is used by ``contextmanager()``, sou you get
  this functionality **automatically**.
* Or, you can write a class that extends from ``ContextDecorator`` as
  a mixin, and implements ``__enter__()``, ``__exit__()``, and ``__call__()``.


.. code:: python

    from contextlib import contextmanager

    @contextmanager
    def feature_flag(name, on=True):
        old_value = feature_flags.is_on(name)
        feature_flags.toggle(name, on)         # behavior of __enter__()
        yield                                  # give up control
        feature_flags.toggle(name, old_value)  # clean up: behavior of __exit__()


* Use it as a context manager

.. code:: python

    with feature_flag(FeatureFlags.SHOW_BETA):
        assert get_homepage_url() == '/beta'

* Or, use as a decorator

.. code:: python

    @feature_flag(FeatureFlags.SHOW_BETA, on=False)
    def get_profile_page():
        bet_flag_on = feature_flags.is_on(FeatureFlags.SHOW_BETA)
        if bet_flag_on:
            return 'beta.html'
        else:
            return 'profile.html'

* Example: FreezeGun - Let your Python tests travel through time.

.. code:: python
    
    import datetime
    from freezgun import freeze_time


    # Use it as a Context Manager
    def test():
        with freeze_time('2012-01-14'):
            assert datetime.datetime.now() == datetime.datetime(2012, 1, 4)
        assert datetime.datetime.now() != datetime.datetime(2012, 1, 4)

    # or a decorator
    @freeze_time('2012-01-14')
    def test():
        assert datetime.datetime.now() == datetime.datetime(2012, 1, 4)


NamedTuple
----------

* Useful when you need lightweight representations of data.
* Create tuple subclasses with named fields.

.. code:: python

    from collections import namedtuple

    CacheInfo = namedtuple('CacheInfo', ['hits', 'misses', 'max_size', 'curr_size'])


* Give NamedTuples default values

.. code:: python

    RoutingRule = namedtuple('RoutingRule', ['prefix, queue_name', 'wait_time'])

    # (1) by specifying defaults
    RoutingRule.__new__.__defaults__ = (None, None, 20)

    # (2) or with _replace to customize a prototype instance
    default_rule = RoutingRule(None, None, 20)
    user_rule = default_rule._replace(prefix='user', queue_name='user-queue')


* NamedTuples can be subclassed and extended

.. code:: python

    class Person(namedtuple('Person', ['first_name', 'last_name'])):
        """Stores first and last name of a Person."""

        def __str__(self):
            return '{} {}'.format(self.first_name, self.last_name)



Summary
-------

* Magic Methods
    * make you objects behave like builtins (numbers, ``list``, ``dict``, etc)

* Method *Magic*
    * aslias methods
    * ``getattr``

* ContextManagers
    * manage resources

* Decorators
    * do something before/after call
    * modify return value
    * validate arguments

* ContextDecorators
    * ContextManagers + Decorators in one

* Iterators and Generators
    * loop over your objects
    * ``yield``

* NamedTuples
    * lightweight classes


Links
-----

* Talk: https://youtu.be/WiQqqB9MlkA
* Slides: https://www.slideshare.net/nnja/elegant-solutions-for-everyday-python-problems-pycon-2018
* https://github.com/mozilla/agithub
* https://github.com/spulec/freezegun
* https://en.wikipedia.org/wiki/Feature_toggle
* Nina Zakharenko on Twitter: `@nnja`_

.. _@nnja: https://twitter.com/nnja

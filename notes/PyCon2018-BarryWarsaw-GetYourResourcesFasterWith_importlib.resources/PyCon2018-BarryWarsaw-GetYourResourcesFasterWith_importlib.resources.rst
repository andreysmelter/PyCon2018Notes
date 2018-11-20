PyCon 2018 - Barry Warsaw - Get your resources faster, with importlib.resources
===============================================================================


Problem
-------

* My code needs some static files.
* How hard can it be to read them at run time?


Types of static files
---------------------

* Templates (e.g. text with placeholders)
* Sample data (e.g. for your tests) 
* Certificates (e.g. connect to http server)
* Internationalization (e.g. gettext translation catalogs)


File system layout
------------------

.. code:: none

    thepkg
    ├── __init__.py
    ├── a.py
    ├── b.py
    └── data
        └── sample.dat

Approaches
----------

Naive approach
~~~~~~~~~~~~~~

.. code:: python

    import thepkg
    from pathlib import Path

    pkg = Path(thepkg.__file__).parent
    path = pkg / 'data' / 'sample.dat'

    with open(path, 'rb') as fp:
        contents = fp.read()


Naive approach: problem
***********************

* Things get complicated if you use archives (e.g. zipfiles and zipapps)
* ``thepkg.__file__`` does not acually point to a real filesystem path.
    
    * you will get ``NotADirectoryError`` exception.


pkg_resources approach
~~~~~~~~~~~~~~~~~~~~~~

.. code:: python

    from pkg_resources import resource_string as resource_bytes

    contents = resource_bytes('thepkg', 'data/sample.dat')


* This works for both file system paths and zip file paths.


pkg_resources approach: problem
*******************************

* Has import-time side-effects (e.g. even if you neverg going to access 
  your sample data you are paying the cost of it).
* It is slow (if you have a lot of things in your sys.path).
* Tries to do too much.
* Has funky APIs (``resource_string`` is really resource bytes).
* It is kind of `everywhere`.


importlib.resources approach
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: python

    from importlib.resources import read_binary

    contents = read_binary('thepkg.data', 'sample.dat')

or

.. code:: python

    from importlib.resources import read_binary
    import thepkg.data

    contents = read_binary('thepkg.data', 'sample.dat')


* In order to do that need to make data directory a package 
  (stick ``__init__.py`` file in it):

.. code:: none

    thepkg
    ├── __init__.py
    ├── a.py
    ├── b.py
    └── data
        ├── __init__.py
        └── sample.dat  


Terminology
-----------

* Access a "resource" in a "package".

.. code:: none

    Q: What's a "package"?
    A: Any importable module with a ``__path__`` attribute.

.. code:: none

    Q: What's a "resource"?
    A: Any readable object contained in a package.

    * Subdirectories/subpackages are not resources!
    * Namespace packages cannot contain resources.


importlib.resource API: High-level API
--------------------------------------

importlib.resource API: Types
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: none

    Package = Union[str, ModuleType]

    Resource = Union[str, os.PathLike]


importlib.resource API: Get the contents of a resource
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* For bytes:

.. code:: none

    read_binary(
        package: Package,
        resource: Resource) -> bytes

* For text:

.. code:: none

    read_text(
        package: Package,
        resource: Resource,
        encoding: str = 'utf-8',
        errors: str = 'strict') -> str


importlib.resource API: Get a file-like object open for reading
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Get a file handle so I can stream bytes out of it.
* Just like builtin ``open`` you can use it with ``with`` statement.


* For binary:

.. code:: none

    open_binary(
        package: Package,
        resource: Resource) -> BinaryIO

* For text:

.. code:: none

    open_text(
        package: Package,
        resource: Resource
        encoding: str = 'utf-8',
        errors: str = 'strict') -> TextIO


importlib.resource API: Get a concrete file system path
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: none

    path(
        package: Package,
        resource: Resource) -> Iterator[Path]


.. code:: none

    with path(thepkg, 'foo.cpython-37m-darwin.so') as lib:
        import_shared_library(lib)

* Unlike with ``pkg_resources`` you now have a guarantee for 
  when that temporary file will get deleted, i.e. as soon as 
  the ``with`` statement exits.


importlib.resource API: List what's in a package
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: none

    contents(
        package: Package) -> Iterable[str]

.. code:: python

    >>> print(sorted(contents('thepkg,data')))
    ['__init__.py', '__pycache__', 'sample.dat']

* Items are not guaranteed to be resources!


importlib.resource API: List resources
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: none

    is_resource(
        package: Package,
        name: str) -> bool


* Use this with ``contents()`` to iterate over resources in a package.



API for loaders: Low-level API
------------------------------

* Low level API for custom loaders
* Built-in support for file system and zips

.. code:: python

    loader.get_resource_reader(
        str: package_name) -> importlib.abc.ResourceReader


* loader - the thing that actually loads the package.
* Any custom loader can play along with higher level API.
* Not limited to file system paths.
* Any loader can import ``get_resource_reader`` and if it exists 
  it gets ``package_name`` and it returns implementation of an ``abc``
  (Abstrac Base Class).


importlib.abc.ResourceReader
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* ``open_resource(str: resource) -> BytesIO``
* ``resource_path(str: resource) -> str``
* ``is_resource(str: name) -> bool``
* ``contents() -> Iterable[str]``

    * ``FileNotFoundError`` raised when resource does not exist.
    * ``resource_path()`` requires a concrete file system path.
    * ``contents()`` can return non-resources.


Performance
-----------

* CLI start up 25-50% faster.
* ``importlib.resources``.
* ``shiv`` (new open source replacement for ``pex``).
* https://shiv.readthedocs.io/en/latest/


importlib_resources
-------------------

* Backport of resource reading for Python 2.7, 3.4-3.6
* https://importlib-resources.readthedocs.io/en/latest/


Links
-----

* Talk: https://youtu.be/ZsGFU2qh73E
* ``shiv``: https://shiv.readthedocs.io/en/latest/
* ``importlib-resources``: https://importlib-resources.readthedocs.io/en/latest/
* Barry Warsaw on Twitter: `@pumpichank`_

.. _@pumpichank: https://twitter.com/pumpichank

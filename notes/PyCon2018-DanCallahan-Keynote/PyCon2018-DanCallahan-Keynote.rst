PyCon 2018 - Dan Callahan Keynote
=================================

* Platform dictates Tool.
* Python is a community that happen to have programming language attached to it.
* There is a sying: "Python is the second best language for anything"
* Python is the first best language in many domains, e.g. science and education.
* The Web is the universal platform.
* Python on the Web?


WEBASSEMBLY
-----------

* New format for programs on the web.
* Low level.
* Binary.
* Complement to JavaScript.
* Gives you virtual CPU inside your browser.


Demo code examples
------------------

Simple recusrive fibonacci function in rust
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: rust

    #[no_mangle]
    pub extern "C" fn fibonacci(n: u32) -> u32 {
        match n {
            0 => 0,
            1 => 1,
            _ => fibonacci(n - 1) + fibonacci(n - 2) 
        }
    }


Build dynamically loaded library
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: none

    $ cargo build --release
    

* Look for ``*.so`` (on linux), ``*.dylib`` (on MacOS), and ``*.dll`` (on Windows).


Use library within Python
~~~~~~~~~~~~~~~~~~~~~~~~~

>>> from cffi import FFI  # python3 -m pip install cffi
>>> ffi = FFI()
>>> ffi.cdef('unsigned int fibonacci(unsigned int);')
>>> demo = ffi.dlopen('libfib')
>>>
>>> [demo.fibonacci(x) for x in range(10)]
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
>>> 


Create wasm binary
~~~~~~~~~~~~~~~~~~

.. code:: none

    $ cargo build --release --target=wasm32-unknown-unknown



* Look for ``*.wasm`` file, then we can feed it to the browser.

.. code:: JavaScript

    >> fetch('demo.wasm')
           .then(response => response.arrayBuffer())
           .then(buffer => WebAssembly.instantiate(buffer))
           .then(result => demo => result.instance.exports)

    >> Array.from(Array(10), keys())
           .map(x => demo.fibonacci(x))


Workflow
~~~~~~~~

* Rust -> dynamically-loaded library -> Python
* Rust -> wasm -> Web

* Python -> wasm -> Web??? 


Conclusions
-----------

* If you have a language that you can compile to the web, 
  then you can run it in the web.
* If we are going to agree on that web is a valuable platform 
  for Python to target, we need to find ways to work alongside 
  the web, and use the web's native capabilities what they are 
  good at and use Python for what Python is good at.

Links
-----

* Talk: https://www.youtube.com/watch?v=ITksU31c1WY
* The birth and death of JavaScript talk: https://www.destroyallsoftware.com/talks/the-birth-and-death-of-javascript
* Dan Callahan on Twitter: @callahad_

.. _@callahad: https://twitter.com/callahad

C to Verilog Compiler
=====================

This section of the manual describes the subset of the C language that is available in *Chips*.

Types
-----

The following types are available in chips:

        + `char`
        + `int`
        + `long`
        + `unsigned char`
        + `unsigned int`
        + `unsigned long`
        + `float`

A `char` is at least 8 bits wide.  An `int` is at least 16 bits wide.  A `long`
is at least 32 bits wide.

The `float` type is implemented as an IEEE 754 single precision floating point
number.

At present, `long long` and `double` have not been implemented, but I
plan to add support for these types in a later release.

single dimensional arrays, `char[]`, `int[]` and `long[]` are supported, but
multidimensional arrays are not yet supported.

`struct` s are supported, you can define arrays of `struct` s, and `struct` s
may contain arrays.

`struct` s cannot yet be passed to a function or returned from a function.

Arrays may be passed (by reference) to functions.

Pointers are not supported, and neither is dynamic memory allocation. This is a
deliberate decision with low memory FPGAs in mind, and probably won't be
supported in future releases.

Functions
---------

Functions are supported. They may be nested, but may not be recursive. Only a
fixed number of arguments is supported, optional arguments are not permitted.

Control Structures
------------------

The following control structures are supported:

+ if/else statements
+ while loop
+ for loop
+ break/continue statements
+ switch/case statements

Operators
---------

The following operators are supported, in order of preference:

+ `()`
+ `~` `-` `!` `sizeof` (unary operators)
+ `*` `/` `%`
+ `+` `-`
+ `<<` `>>`
+ `<` `>` `<=` `>=`
+ `==` `!=`
+ `&`
+ '^`
+ `|`
+ `&&`
+ `||`
+ \`? : `


Stream I/O
----------

The language has been extended to allow components to communicate by sending
data through streams.

Stream I/O is achieved by calling built-in functions with special names.
Functions that start with the name `input` or `output` are interpreted as "read
from input", or "write to output".

.. code-block:: c

    int temp;
    temp = input_spam(); //reads from an input called spam
    temp = input_eggs(); //reads from an input called eggs
    output_fish(temp);   //writes to an output called fish

Reading or writing from inputs and outputs causes program execution to block
until data is available. If you don't want to commit yourself to reading and
input and blocking execution, you can check if data is ready.

.. code-block:: c

    int temp;
    if(ready_spam()){
       temp = input_spam();
    }

There is no equivalent function to check if an output is ready to receive data,
this could cause deadlocks if both the sending and receiving end were waiting
for one another. 

Timed Waits
-----------

Timed waits can be achieved using the built-in `wait-clocks` function. The
wait_clocks function accepts a single argument, the numbers of clock cycles to
wait.

.. code-block:: c
    
    wait_clocks(100); //wait for 1 us with 100MHz clock


Debug and Test
--------------

The built in `report` function displays the value of an expression in the
simulation console. This will have no effect in a synthesised design.

.. code-block:: c

    int temp = 4;
    report(temp); //prints 4 to console
    report(10); //prints 10 to the console


The built in function assert causes a simulation error if it is passed a zero
value. The assert function has no effect in a synthesised design.

.. code-block:: c

    int temp = 5;
    assert(temp); //does not cause an error
    int temp = 0;
    assert(temp); //will cause a simulation error
    assert(2+2==5); //will cause a simulation error

In simulation, you can write values to a file using the built-in `file_write`
function. The first argument is the value to write, and the second argument is
the file to write to. The file will be overwritten when the simulation starts,
and subsequent calls will append a new vale to the end of the file. Each value
will appear in decimal format on a separate line. A file write has no effect in
a synthesised design.

.. code-block:: c

    file_write(1, "simulation_log.txt");
    file_write(2, "simulation_log.txt");
    file_write(3, "simulation_log.txt");
    file_write(4, "simulation_log.txt");

You can also read values from a file during simulation. A simulation error will
occur if there are no more value in the file.

.. code-block:: c

    assert(file_read("simulation_log.txt") == 1);
    assert(file_read("simulation_log.txt") == 2);
    assert(file_read("simulation_log.txt") == 3);
    assert(file_read("simulation_log.txt") == 4);


C Preprocessor
--------------

The C preprocessor currently has only limited capabilities, and currently only
the `#include` feature is supported.

c2verilog
---------

For simple designs with only one C component, the simplest way to generate Verilog is by using the c2verilog utility.
The utility accepts C files as input, and generates Verilog files as output.

::

    ~$ c2verilog input_file.c

You may automatically compile the output using Icarus Verilog by adding the
`iverilog` option. You may also run the Icarus Verilog simulation using the
`run` option.

::

    ~$ c2verilog iverilog run input_file.c

You can also influence the way the Verilog is generated. By default, a low area
solution is implemented. If you can specify a design optimised for speed using
the `speed` option.


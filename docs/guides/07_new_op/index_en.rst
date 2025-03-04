###################
Write New Operators
###################

This section will guide you on how to use the custom operator mechanism of Paddle, including the following two categories:

1. C++ operator: The writing method is relatively simple, does not involve the internal concept of the framework, does not need to recompile the paddle framework, and is used as an external module.
2. Python operator: use Python to implement forward and backward methods, then used in network.

- `Custom C++ Operator <./new_custom_op_cn.html>`_

- `Custom Python Operator <./new_python_op_cn.html>`_

- `Kernel Primitives API <./kernel_primitive_api/index_en.html>`_ : Introduce the block-level CUDA functions provided by PaddlePaddle to speed up operator development.

.. toctree::
   :hidden:

   new_op_en.md
   op_notes_en.md
   kernel_primitive_api/index_en.rst

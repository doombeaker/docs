.. _cn_api_tensor_ones:

ones
-------------------------------

.. py:function:: paddle.ones(shape, dtype=None)



创建形状为 ``shape`` 、数据类型为 ``dtype`` 且值全为1的Tensor。

参数
:::::::::

    - **shape** (tuple|list|Tensor) - 输出Tensor的形状， ``shape`` 的数据类型为int32或者int64。
    - **dtype** (np.dtype|str， 可选) - 输出Tensor的数据类型，数据类型必须为bool、 float16、float32、float64、int32或int64。如果 ``dtype`` 为None，默认数据类型为float32。
    - **name** （str，可选）- 具体用法请参见 :ref:`api_guide_Name` ，一般无需设置，默认值为None。

返回
:::::::::
值全为1的Tensor，数据类型和 ``dtype`` 定义的类型一致。


代码示例
:::::::::

.. code-block:: python

    
    import paddle

    #default dtype for ones OP
    data1 = paddle.ones(shape=[3, 2]) 
    # [[1. 1.]
    #  [1. 1.]
    #  [1. 1.]]
    data2 = paddle.ones(shape=[2, 2], dtype='int32') 
    # [[1 1]
    #  [1 1]]

    #attr shape is a Tensor
    shape = paddle.full(shape=[2], dtype='int32', fill_value=2)
    data3 = paddle.ones(shape=shape, dtype='int32') 
    # [[1 1]
    #  [1 1]]


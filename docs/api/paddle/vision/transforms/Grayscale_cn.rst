.. _cn_api_vision_transforms_Grayscale:

Grayscale
-------------------------------

.. py:class:: paddle.vision.transforms.Grayscale(num_output_channels=1, keys=None)

将图像转换为灰度。

参数
:::::::::

    - num_output_channels (int，可选) - 输出图像的通道数，参数值为1或3。默认值：1。
    - keys (list[str]|tuple[str]，可选) - 与 ``BaseTransform`` 定义一致。默认值: None。

形状
:::::::::

    - img (PIL.Image|np.ndarray|Paddle.Tensor) - 输入的图像数据，数据格式为'HWC'。
    - output (PIL.Image|np.ndarray|Paddle.Tensor) - 返回输入图像的灰度版本。如果 output_channels == 1，返回一个单通道图像。如果 output_channels == 3，返回一个3通道图像，其中RGB三个通道值一样。

返回
:::::::::

    计算 ``Grayscale`` 的可调用对象。

代码示例
:::::::::
    
.. code-block:: python

    import numpy as np
    from PIL import Image
    from paddle.vision.transforms import Grayscale

    transform = Grayscale()

    fake_img = Image.fromarray((np.random.rand(224, 224, 3) * 255.).astype(np.uint8))

    fake_img = transform(fake_img)
    print(np.array(fake_img).shape)
    
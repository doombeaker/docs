
# 2.3.0-rc0 Release Note

## 1. 重要更新

我们很高兴地发布飞桨框架 2.3.0-rc0 版本，本版本包含如下重要更新。

### API

- 新增 100 多个 API，覆盖自动微分、线性代数、概率分布、稀疏张量、框架性能分析、硬件设备管理、视觉领域等方面。

- 新增 4 个自动微分 API，11 个线性代数 API，21 个概率分布类 API，更好地支持科学计算、强化学习等场景。

- 新增 11 个 稀疏张量计算 API，支持创建 COO、CRS 格式的 Sparse Tensor 以及与 Tensor 互相转换等基础功能。

- 新增 9 个框架性能分析 API，以`paddle.profiler.Profiler`为核心，提供对训练、推理过程中性能数据的收集、导出和统计的功能。

- 新增 7 个硬件设备管理 API，更好支持硬件相关信息获取。

- 新增多个视觉、文本领域 API，方便复用 MobileNetV3, ResNeXt等骨干网络，实现快速组网。

### 飞桨高可复用算子库 PHI

- 发布飞桨高可复用算子库 PHI (Paddle HIgh reusability operator library)，支持组合式算子功能复用、Primitive算子内核复用、插件式硬件加速库复用。针对飞桨框架原算子库存在的算子接口不清晰、算子复用成本较高、调用性能不够快的问题，我们重构了飞桨框架的算子库，设计了灵活、高效的函数式算子库 Phi，可以通过对函数式算子接口组合调用的方式实现新算子。新算子库提供了 200 余个跟 python 开发接口保持一致的 C++ 运算类 API，以及近500个可供组合调用的前、反向函数式算子内核 Kernel，可大幅降低框架原生算子和自定义算子的开发成本。新算子库支持Primitive API方式开发算子内核，可支持不同硬件（比如GPU和XPU）的算子内核复用。新算子库支持以插件方式接入硬件（比如NPU）的加速库，实现低成本复用硬件加速库。

### 分布式训练

- 全面升级自适应分布式训练架构，含弹性扩缩容、异步流水执行器、异构通信、自动并行等多个模块，支持了多种异构硬件下自动感知的分布式训练及分布式推理。

- 动态图混合并行下新增MoE并行策略、GroupSharded 并行策略、Pure FP16 等，进一步支持了动态图下大模型的高效并行训练。

- 全面升级优化了通用异构参数服务器架构，进行各模块的抽象简化，如通信、存储等，提升了参数服务器的二次开发体验；GPU 参数服务器在千亿参数百亿数据分钟级流式训练下性能提升2.38倍。

### 编译安装

- 从 2.3.0-rc0 版本开始，飞桨对框架支持的 GPU 架构种类进行了调整和升级。

### 推理部署

- 新增 Java API 和 ONNX Runtime CPU 后端。

- 支持 TensorRT 8.0 / 8.2 和结构化稀疏，针对 ERNIE 类结构模型性能深度优化。

### 硬件适配

- 新增自定义新硬件接入：提供一种插件式扩展 PaddlePaddle 硬件后端的方式。

- 新增对华为昇腾910 / GraphCore IPU / 寒武纪MLU / 昆仑芯2代多种异构芯片的训练/推理支持。

### 框架架构

- 这个版本中，我们在框架的执行器也做了大量工作，详情请见：[新动态图执行机制](#%E6%96%B0%E5%8A%A8%E6%80%81%E5%9B%BE%E6%89%A7%E8%A1%8C%E6%9C%BA%E5%88%B6) 与 [全新静态图执行器](#%E5%85%A8%E6%96%B0%E9%9D%99%E6%80%81%E5%9B%BE%E6%89%A7%E8%A1%8C%E5%99%A8)。

## 2. 不兼容升级

- `paddle.to_tensor` 将一个 python int scalar 转换为 Tensor 时，在 Windows 上的默认数据类型由 int32 变为 int64，从而与 Linux/Mac 保持对齐。([#39662](https://github.com/PaddlePaddle/Paddle/pull/39662)) 

- 为了与 python3 下的除法行为保持一致，除法符号 `/` 从 rounding divide 变成 true divide，计算输出结果的数据类型从 int 切换成 float。 ([#40890](https://github.com/PaddlePaddle/Paddle/pull/40890)) 

<table>
<tr>
<th>
2.2
</th>
<th>
2.3.0-rc0
</th>
</tr>

<tr>
<td>
<pre>

```python
>>> import paddle
>>> a = paddle.to_tensor([327])
>>> b = paddle.to_tensor([80])
>>> a / b
Tensor(shape=[1], dtype=int64, place=CUDAPlace(0), stop_gradient=True,
      [4])
```
</pre>
</td>
<td>
<pre>

```python
>>> import paddle
>>> a = paddle.to_tensor([327])
>>> b = paddle.to_tensor([80])
>>> a / b
Tensor(shape=[1], dtype=float32, place=Place(gpu:0), stop_gradient=True,
      [4.08750010])
```
</pre>
</td>
</tr>
</table>

- 修正 ELU 的公式，alpha < 0 时的计算方式与原论文对齐，从而修复小部分情况下的计算结果错误。同时，由于在 alpha < 0 无法在数学上仅从输出计算反向梯度，因此 elu_ 在 alpha < 0 时将报错。([#37316](https://github.com/PaddlePaddle/Paddle/pull/37316))

<table>
<tr>
<th>
2.2
</th>
<th>
2.3.0-rc0
</th>
</tr>

<tr>
<td>
<pre>

```python
# elu(x) = max(0, x) + min(0, α ∗ (e^x − 1))
>>> import paddle
>>> x = paddle.to_tensor([-1. ,6.])
>>> m = paddle.nn.ELU(-0.2)
>>> out = m(x)
>>> out
Tensor(shape=[2], dtype=float32, place=CUDAPlace(0), stop_gradient=True,
       [ 0.         , -74.48576355])
>>> out = paddle.nn.functional.elu_(x, alpha=-0.2, name=None)
>>> out
Tensor(shape=[2], dtype=float32, place=CUDAPlace(0), stop_gradient=True,
       [ 0.         , -74.48576355])
```
</pre>
</td>
<td>
<pre>

```python
# elu(x) = x, if x > 0
# elu(x) = α ∗ (e^x − 1), if x <= 0
>>> import paddle
>>> x = paddle.to_tensor([-1. ,6.])
>>> m = paddle.nn.ELU(-0.2)
>>> out = m(x)
>>> out
Tensor(shape=[2], dtype=float32, place=CUDAPlace(0), stop_gradient=True,
       [0.12642412,  6.        ])
>>> out = paddle.nn.functional.elu_(x, alpha=-0.2, name=None)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/local/lib/python3.7/dist-packages/decorator.py", line 232, in fun
    return caller(func, *(extras + args), **kw)
  File "/usr/local/lib/python3.7/dist-packages/paddle/fluid/wrapped_decorator.py", line 25, in __impl__
    return wrapped_func(*args, **kwargs)
  File "/usr/local/lib/python3.7/dist-packages/paddle/fluid/dygraph/inplace_utils.py", line 34, in __impl__
    return func(*args, **kwargs)
  File "/usr/local/lib/python3.7/dist-packages/paddle/nn/functional/activation.py", line 89, in elu_
    assert alpha >= 0., "elu_ only support alpha >= 0, please use elu instead."
AssertionError: elu_ only support alpha >= 0, please use elu instead.
```
</pre>
</td>
</tr>
</table>

## 3. 训练框架（含分布式）

### （1）新功能

#### API

- 新增4个自动微分类 API，支持科学计算需求，具体列表如下：([#40692](https://github.com/PaddlePaddle/Paddle/pull/40692))
  
  - `paddle.incubate.autograd.vjp`，计算向量-雅可比矩阵乘积。
  
  - `paddle.incubate.autograd.jvp`，计算雅可比矩阵-向量乘积。
  
  - `paddle.incubate.autograd.Jacobian`，计算雅可比矩阵。
  
  - `paddle.incubate.autograd.Hessian`，计算海森矩阵。

- 新增线性代数类 API
  
  - 新增 `paddle.linalg.triangular_solve`，计算具有唯一解的三角系数线性方程组。([#36714](https://github.com/PaddlePaddle/Paddle/pull/36714)) 
  
  - 新增 `paddle.linalg.eig`，计算一般方阵的特征分解。([#35764](https://github.com/PaddlePaddle/Paddle/pull/35764)) 
  
  - 新增 `paddle.linalg.sovle`，计算线性方程组的解。([#35715](https://github.com/PaddlePaddle/Paddle/pull/35715))
  
  - 新增 `paddle.linalg.lstsq`，计算线性方程组的最小二乘解。([#38585](https://github.com/PaddlePaddle/Paddle/pull/38585), [#38621](https://github.com/PaddlePaddle/Paddle/pull/38621)) 
  
  - 新增 `paddle.linalg.qr`，计算矩阵的 QR 分解。([#35742](https://github.com/PaddlePaddle/Paddle/pull/35742), [#38824](https://github.com/PaddlePaddle/Paddle/pull/38824)）
  
  - 新增 `paddle.inner`，计算矩阵内积。([#37706](https://github.com/PaddlePaddle/Paddle/pull/37706)) 
  
  - 新增 `paddle.outer`，计算矩阵外积。([#37706](https://github.com/PaddlePaddle/Paddle/pull/37706)) 
  
  - 新增 `paddle.linalg.cov`，计算向量间协方差。([#38392](https://github.com/PaddlePaddle/Paddle/pull/38392)) 
  
  - 新增 `paddle.linalg.cholesky_sovle`，计算方程 cholesky 解。([#38167](https://github.com/PaddlePaddle/Paddle/pull/38167)) 
  
  - 新增 `paddle.linalg.lu`、 `paddle.linalg.lu_unpack`，计算矩阵 lu 分解、解压缩 lu 矩阵。([#38617](https://github.com/PaddlePaddle/Paddle/pull/38617), [#38559](https://github.com/PaddlePaddle/Paddle/pull/38559), [#38616](https://github.com/PaddlePaddle/Paddle/pull/38616)) 

- 新增21个概率分布类 API，包括6个随机变量分布，13个随机变量变换，2个 KL 散度计算，用于强化学习、变分推断、科学计算等场景，具体列表如下：([#40536](https://github.com/PaddlePaddle/Paddle/pull/40536), [#38820](https://github.com/PaddlePaddle/Paddle/pull/38820), [#38558](https://github.com/PaddlePaddle/Paddle/pull/38558/files), [#38445](https://github.com/PaddlePaddle/Paddle/pull/38445), [#38244](https://github.com/PaddlePaddle/Paddle/pull/38244), [#38047](https://github.com/PaddlePaddle/Paddle/pull/38047))
  
  - `paddle.distribution.ExponentialFamily`，指数分布族基类。
  
  - `paddle.distribution.Beta`，`Beta` 分布。
  
  - `paddle.distribution.Dirichlet`，`Dirichlet` 分布。
  
  - `paddle.distribution.Independent`，独立分布，用于创建高阶分布。
  
  - `paddle.distribution.TransformedDistribution`，变换分布，用于通过基础分布及一系列变换生成高阶分布。
  
  - `paddle.distribution.Multionmial`，多项分布。
  
  - `paddle.distribution.Transform`，随机变量变换的基类。
  
  - `paddle.distribution.AbsTransform`，取绝对值变换。
  
  - `paddle.distribution.AffineTransform`，仿射变换。
  
  - `paddle.distribution.ChainTransform`，变换的链式组合。
  
  - `paddle.distribution.ExpTransform`，指数变换。
  
  - `paddle.distribution.IndependentTransform`，独立变换，用于扩展变换定义域的 `event_dim`。
  
  - `paddle.distribution.PowerTransform`，幂变换。
  
  - `paddle.distribution.ReshapeTransform`，`reshape` 变换。
  
  - `paddle.distribution.SigmoidTransform`，`sigmoid` 变换。
  
  - `paddle.distribution.SoftmaxTransform`，`softmax` 变换。
  
  - `paddle.distribution.StackTransform`，`stack` 变换，用于以 `stack` 方式组合多个变换。
  
  - `paddle.distribution.StickBreakingTransform` , `stickbreaking` 变换。
  
  - `paddle.distribution.TanhTransform`，`tanh` 变换。
  
  - `paddle.distribution.kl_divergence`，计算 KL 散度。
  
  - `paddle.distribution.register_kl`，注册用户自定义 KL 散度计算函数。

- 新增高层 API
  
  - 新增 `paddle.vision.models.AlexNet`、`paddle.vision.models.alexnet`，支持直接使用 AlexNet 模型。([#36058](https://github.com/PaddlePaddle/Paddle/pull/36058)) 
  
  - 新增 `paddle.vision.models.DenseNet`、 `paddle.vision.models.densenet121`、 `paddle.vision.models.densenet161`、 `paddle.vision.models.densenet169`、 `paddle.vision.models.densenet201`、 `paddle.vision.models.densenet264`，支持直接使用 DenseNet 模型。([#36069](https://github.com/PaddlePaddle/Paddle/pull/36069)) 
  
  - 新增 `paddle.vision.models.GoogLeNet`、`paddle.vision.models.googlenet`，支持直接使用 GoogLeNet 模型。([#36034](https://github.com/PaddlePaddle/Paddle/pull/36034)) 
  
  - 新增 `paddle.vision.models.InceptionV3`、`paddle.vision.models.inception_v3`，支持直接使用 InceptionV3 模型。([#36064](https://github.com/PaddlePaddle/Paddle/pull/36064)) 
  
  - 新增 `paddle.vision.models.MobileNetV3Small`、 `paddle.vision.models.MobileNetV3Large`、`paddle.vision.models.mobilenet_v3_small`、`paddle.vision.models.mobilenet_v3_large`，支持直接使用 MobileNetV3 模型。([#38653](https://github.com/PaddlePaddle/Paddle/pull/38653)) 
  
  - 新增 `paddle.vision.models.ResNeXt`、 `paddle.vision.models.resnext50_32x4d`、 `paddle.vision.models.resnext50_64x4d`、`paddle.vision.models.resnext101_32x4d`、`paddle.vision.models.resnext101_64x4d`、`paddle.vision.models.resnext152_32x4d`、`paddle.vision.models.resnext152_64x4d`，支持直接使用 ResNeXt 模型。([#36070](https://github.com/PaddlePaddle/Paddle/pull/36070)) 
  
  - 新增 `paddle.vision.models.ShuffleNetV2`、 `paddle.vision.models.shufflenet_v2_x0_25`、`paddle.vision.models.shufflenet_v2_x0_33`、`paddle.vision.models.shufflenet_v2_x0_5`、`paddle.vision.models.shufflenet_v2_x1_0`、`paddle.vision.models.shufflenet_v2_x1_5`、`paddle.vision.models.shufflenet_v2_x2_0`、`paddle.vision.models.shufflenet_v2_swish`，支持直接使用 ShuffleNetV2 模型。([#36067](https://github.com/PaddlePaddle/Paddle/pull/36067)) 
  
  - 新增 `paddle.vision.models.SqueezeNet`、 `paddle.vision.models.squeezenet1_0`、`paddle.vision.models.squeezenet1_1`，支持直接使用 SqueezeNet 模型。([#36066](https://github.com/PaddlePaddle/Paddle/pull/36066)) 
  
  - 新增 `paddle.vision.models.wide_resnet50_2`、`paddle.vision.models.wide_resnet101_2`，支持直接使用 WideResNet 模型。([#36952](https://github.com/PaddlePaddle/Paddle/pull/36952))
  
  - 新增`paddle.vision.ops.nms` API，支持单类别和多类别非极大抑制(non-maximum supression, nms)算法，用于目标检测预测任务加速。([#40962](https://github.com/PaddlePaddle/Paddle/pull/40962)) 
  
  - 新增`paddle.vision.ops.roi_pool` 和 `paddle.vision.ops.RoIPool`，支持检测任务中 RoI 区域池化操作。 ([#36154](https://github.com/PaddlePaddle/Paddle/pull/36154)) 
  
  - 新增`paddle.vision.ops.roi_align` 和 `paddle.vision.ops.RoIAlign`，支持检测任务中 RoI Align 操作。 ([#35102](https://github.com/PaddlePaddle/Paddle/pull/36154)) 
  
  - 新增 `paddle.text.ViterbiDecoder`、`paddle.text.viterbi_decode` Viterbi 解码 API，主要用于序列标注模型的预测。 ([#35778](https://github.com/PaddlePaddle/Paddle/pull/35778)) 

- 新增 11 个 Sparse 类 API，支持创建 COO、CRS 格式的Sparse Tensor，与 Tensor 互相转换等基础功能： 
  
  - `paddle.sparse.sparse_coo_tensor`，创建 COO 格式的 Sparse Tensor。 ([#40780](https://github.com/PaddlePaddle/Paddle/pull/40780)）
  
  - `paddle.sparse.sparse_csr_tensor`，创建 CSR 格式的 Sparse Tensor。 ([#40780](https://github.com/PaddlePaddle/Paddle/pull/40780)）
  
  - `paddle.sparse.ReLU`，支持 SparseCooTensor 的 ReLU 激活层。（[#40959](https://github.com/PaddlePaddle/Paddle/pull/40959)) 
  
  - `paddle.sparse.functional.relu`，支持 SparseCooTensor 的 ReLU 函数。（[#40959](https://github.com/PaddlePaddle/Paddle/pull/40959)) 
  
  - `Tensor.values()`，获取 SparseCooTensor 或者 SparseCsrTensor 的非零元素方法。（[#40608](https://github.com/PaddlePaddle/Paddle/pull/40608)）
  
  - `Tensor.indices()`，获取 SparseCooTensor 的坐标信息的方法。（[#40608](https://github.com/PaddlePaddle/Paddle/pull/40608)）
  
  - `Tensor.crows()`，获取 SparseCsrTensor 的压缩行信息的方法。（[#40608](https://github.com/PaddlePaddle/Paddle/pull/40608)）
  
  - `Tensor.cols()`，获取 SparseCsrTensor 的列信息的方法。（[#40608](https://github.com/PaddlePaddle/Paddle/pull/40608)）
  
  - `Tensor.to_sparse_coo()`，将 DenseTensor 或者 SparseCsrTensor 转换为 SparseCooTensor。 ([#40780](https://github.com/PaddlePaddle/Paddle/pull/40780)）
  
  - `Tensor.to_sparse_csr()`，将 DenseTensor 或者 SparseCooTensor 转换为 SparseCsrTensor。([#40780](https://github.com/PaddlePaddle/Paddle/pull/40780)）
  
  - `Tensor.to_dense()`，将 SparseCooTensor 或者 SparseCsrTensor 转换为 DenseTensor。([#40780](https://github.com/PaddlePaddle/Paddle/pull/40780)）

- 新增硬件相关 API
  
  - 新增 `paddle.device.cuda.max_memory_allocated`、`paddle.device.cuda.max_memory_reserved`、 `paddle.device.cuda.memory_allocated` 和 `paddle.device.cuda.memory_reserved` 四个 GPU 显存监测相关 API，方便实时查看和分析模型显存占用指标。([#38657](https://github.com/PaddlePaddle/Paddle/pull/38657)) 
  
  - 新增 `paddle.device.cuda.get_device_properties`，支持返回 CUDA 设备属性信息。([#35661](https://github.com/PaddlePaddle/Paddle/pull/35661)) 
  
  - 新增 `paddle.device.cuda.get_device_name` 和 `paddle.device.cuda.get_device_capability`，支持返回 GPU 设备名称信息和计算能力的主要和次要修订号。([#35672](https://github.com/PaddlePaddle/Paddle/pull/35672)) 

- 新增 Tensor 操作 API
  
  - 新增 `paddle.nansum`，沿 `axis` 对输入 Tensor 求和，且忽略掉 `NaNs` 值。([#38137](https://github.com/PaddlePaddle/Paddle/pull/38137)) 
  
  - 新增 `paddle.nanmean`，沿 `axis`对输入 Tensor 求平均，且忽略掉 `NaNs` 值。（[#40472](https://github.com/PaddlePaddle/Paddle/pull/40472)）
  
  - 新增 `paddle.clone`，返回输入 Tensor 的拷贝，并且提供梯度计算。([#38020](https://github.com/PaddlePaddle/Paddle/pull/38020)) 
  
  - 新增 `paddle.Tensor.element_size`，返回 Tensor 中的单个元素在计算机中所分配的 bytes 数量。([#38020](https://github.com/PaddlePaddle/Paddle/pull/38020)) 
  
  - 新增 `paddle.Tensor.to_uva_tensor`，支持将 numpy 对象转换为实际存储在 CPU，但可作为 CUDA 对象进行虚拟地址访问的功能。([#39146](https://github.com/PaddlePaddle/Paddle/pull/39146), [#38950](https://github.com/PaddlePaddle/Paddle/pull/38950)) 
  
  - 新增`paddle.rot90`，沿 `axes` 指定的平面将 n 维 Tensor 旋转90度。([#37634](https://github.com/PaddlePaddle/Paddle/pull/37634)) 
  
  - 新增`paddle.logit` 和 `paddle.Tensor.logit`，计算输入 Tensor 的 logit 函数值。([#37844](https://github.com/PaddlePaddle/Paddle/pull/37844)) 
  
  - 新增 `paddle.repeat_interleave`，沿着指定轴对输入进行复制，创建并返回到一个新的 Tensor。([#37981](https://github.com/PaddlePaddle/Paddle/pull/37981)) 
  
  - 新增 `paddle.renorm`，把 Tensor 在指定的 `axis` 切分成多块后分别进行 p norm 操作。([#38130](https://github.com/PaddlePaddle/Paddle/pull/38130), [#38459](https://github.com/PaddlePaddle/Paddle/pull/38459)) 
  
  - 新增 `paddle.mode` 和 `paddle.Tensor.mode`，沿指定轴查找输入 Tensor 的众数及对应的索引。([#38446](https://github.com/PaddlePaddle/Paddle/pull/38446)) 
  
  - 新增 `paddle.quantile` 和 `paddle.Tensor.quantile`，沿指定轴计算 Tensor 的 q 分位数。([#38567](https://github.com/PaddlePaddle/Paddle/pull/38567)) 
  
  - 新增 `paddle.kthvalue` 和 `paddle.Tensor.kthvalue`，查找 Tensor 中指定轴上第 k 小的数及对应的索引。([#38386](https://github.com/PaddlePaddle/Paddle/pull/38386)) 
  
  - 新增 `paddle.is_floating_point` 和 `paddle.Tensor.is_floating_point`，判断输入 Tensor 是否为浮点类型。([#37885](https://github.com/PaddlePaddle/Paddle/pull/37885)) 
  
  - 新增 `paddle.erfinv` 和 `paddle.Tensor.erfinv`，计算输入 Tensor 的逆误差函数。([#38295](https://github.com/PaddlePaddle/Paddle/pull/38295)) 
  
  - 新增 `paddle.lerp` 和 `paddle.Tensor.lerp`，根据给定权重计算输入Tensor间的线性插值。([#37253](https://github.com/PaddlePaddle/Paddle/pull/37253)) 
  
  - 新增 `paddle.angle`，用于计算复数 Tensor 的相位角。 ([#37689](https://github.com/PaddlePaddle/Paddle/pull/37689)) 
  
  - 新增`paddle.rad2deg`和`paddle.Tensor.rad2deg`，将元素从弧度的角度转换为度。([#37598](https://github.com/PaddlePaddle/Paddle/pull/37598)) 
  
  - 新增`paddle.deg2rad`和`paddle.Tensor.deg2rad`，将元素从度的角度转换为弧度。([#37598](https://github.com/PaddlePaddle/Paddle/pull/37598)) 
  
  - 新增`paddle.gcd`和`paddle.Tensor.gcd`，计算两个输入的按元素绝对值的最大公约数。([#37819](https://github.com/PaddlePaddle/Paddle/pull/37819)) 
  
  - 新增`paddle.lcm`和`paddle.Tensor.lcm`，计算两个输入的按元素绝对值的最小公倍数。([#37819](https://github.com/PaddlePaddle/Paddle/pull/37819)) 
  
  - 新增`paddle.amax`和`paddle.Tensor.amax`，对指定维度上的 Tensor 元素求最大值，正向结果和 max 一样，有多个相等的最大值时，反向的梯度平均分到这多个值的位置上。([#38417](https://github.com/PaddlePaddle/Paddle/pull/38417)) 
  
  - 新增`paddle.amin`和`paddle.Tensor.amin`，对指定维度上的 Tensor 元素求最小值，正向结果和 min 一样，有多个相等的最小值时，反向的梯度平均分到这多个值的位置上。([#38417](https://github.com/PaddlePaddle/Paddle/pull/38417)) 
  
  - 新增`paddle.isclose`，用于判断两个 Tensor 的每个元素是否接近。([#37135](https://github.com/PaddlePaddle/Paddle/pull/37135)) 
  
  - 新增`paddle.put_along_axis` 和`paddle.take_along_axis`，用于提取或放置指定索引下标的元素。([#38608](https://github.com/PaddlePaddle/Paddle/pull/38608)) 
  
  - 新增 `paddle.bincount` 和 `paddle.Tensor.bincount`，用于统计 Tensor 中每个元素出现的次数。([#36317](https://github.com/PaddlePaddle/Paddle/pull/36317)) 
  
  - 新增 `paddle.fmax`、 `paddle.fmin`，扩展了max/min的功能，支持比较的两个 Tensor 中有 NaN 值的情况，即如果对应位置上有1个 NaN 值，则返回那个非 NaN 值；如果对应位置上有2个 NaN 值，则返回 NaN 值。([#37826](https://github.com/PaddlePaddle/Paddle/pull/37826)) 
  
  - 新增 `paddle.diff`，用于计算沿给定维度的第 n 个前向差值，目前支持 n=1。([#37441](https://github.com/PaddlePaddle/Paddle/pull/37441)) 
  
  - 新增 `paddle.asinh`、`paddle.acosh`、`paddle.atanh` 反双曲函数类 API。 ([#37076](https://github.com/PaddlePaddle/Paddle/pull/37076)) 
  
  - 新增 `paddle.as_real`，`paddle.as_complex` 用于实数 Tensor 和复数 Tensor 之间的转换。 ([#37784](https://github.com/PaddlePaddle/Paddle/pull/37784)) 
  
  - 新增 `paddle.complex` 用于给定实部和虚部构造复数 Tensor。 ([#37918](https://github.com/PaddlePaddle/Paddle/pull/37918), [#38272](https://github.com/PaddlePaddle/Paddle/pull/38272)) 
  
  - 新增 `paddle.det` 与 `paddle.slogdet`，用于计算矩阵的行列式和行列式的自然对数。 ([#34992](https://github.com/PaddlePaddle/Paddle/pull/34992)) 
  
  - 新增`paddle.nn.utils.parameters_to_vector`，可以将输入的多个 parameter 展平并连接为1个1-D Tensor。([#38020](https://github.com/PaddlePaddle/Paddle/pull/38020)) 
  
  - 新增`paddle.nn.utils.vector_to_parameters`，将1个1-D Tensor按顺序切分给输入的多个 parameter。([#38020](https://github.com/PaddlePaddle/Paddle/pull/38020)) 

- 新增组网类 API
  
  - 新增 `paddle.nn.Fold`、`paddle.nn.functional.fold`，支持将提取出的滑动局部区域块还原成 batch 的 Tensor。([#38613](https://github.com/PaddlePaddle/Paddle/pull/38613)) 
  
  - 新增 `paddle.nn.CELU`、`paddle.nn.functional.celu`，支持 CELU 激活层。([#36088](https://github.com/PaddlePaddle/Paddle/pull/36088)) 
  
  - 新增 `paddle.nn.HingeEmbeddingLoss`，增加计算 hinge embedding 损失的方式，通常用于学习 nonlinear embedding 或半监督学习。([#37540](https://github.com/PaddlePaddle/Paddle/pull/37540))
  
  - 新增 `paddle.nn.ZeroPad2D` API，按照 padding 属性对输入进行零填充。([#37151](https://github.com/PaddlePaddle/Paddle/pull/37151))
  
  - 新增 `paddle.nn.MaxUnPool3D` 和 `paddle.nn.MaxUnPool1D`，用于计算 3D 最大反池化和 1D 最大反池化。([#38716](https://github.com/PaddlePaddle/Paddle/pull/38716)) 
  
  - 新增 `paddle.incubate.graph_khop_sampler`、`paddle.incubate.graph_sample_neighbors`、 `paddle.incubate.graph_reindex` API，支持图多阶邻居采样和图编号重索引操作，主要用于图神经网络模型训练。([#39146](https://github.com/PaddlePaddle/Paddle/pull/39146), [#40809](https://github.com/PaddlePaddle/Paddle/pull/40809)) 

- 新增随机数类 API
  
  - 新增 `paddle.poisson`，以输入 Tensor 为泊松分布的 lambda 参数，生成一个泊松分布的随机数 Tensor。([#38117](https://github.com/PaddlePaddle/Paddle/pull/38117)) 
  
  - 新增 `paddle.randint_like` API，支持新建服从均匀分布的、范围在[low, high) 的随机 Tensor，输出的形状与输入的形状一致。([#36169](https://github.com/PaddlePaddle/Paddle/pull/36169)) 
  
  - 新增 `paddle.Tensor.exponential_`，为 inplace 式 API，通过指数分布随机数来填充输入 Tensor。([#38256](https://github.com/PaddlePaddle/Paddle/pull/38256)) 

- 新增参数初始化类 API
  
  - 新增`paddle.nn.initializer.Dirac`，通过迪拉克 delta 函数来初始化 3D/4D/5D 参数，其常用于卷积层 Conv1D/Conv2D/Conv3D 的参数初始化。([#37389](https://github.com/PaddlePaddle/Paddle/pull/37389)) 
  
  - 新增`paddle.nn.initializer.Orthogonal`，正交矩阵初始化，被初始化后的参数是（半）正交向量。([#37163](https://github.com/PaddlePaddle/Paddle/pull/37163)) 
  
  - 新增`paddle.nn.initializer.calculate_gain`，获取激活函数的推荐增益值，增益值可用于设置某些初始化 API，以调整初始化范围。([#37163](https://github.com/PaddlePaddle/Paddle/pull/37163)) 

- 新增学习率类 API
  
  - 新增 `paddle.optimizer.lr.MultiplicativeDecay`，提供 `lambda` 函数设置学习率的策略。([#38250](https://github.com/PaddlePaddle/Paddle/pull/38250))

- 新增分布式相关 API
  
  - 新增 `paddle.incubate.optimizer.DistributedFusedLamb`，使得 Lamb 优化器可分布式更新参数。([#40011](https://github.com/PaddlePaddle/Paddle/pull/40011), [#39972](https://github.com/PaddlePaddle/Paddle/pull/39972), [#39900](https://github.com/PaddlePaddle/Paddle/pull/39900), [#39747](https://github.com/PaddlePaddle/Paddle/pull/39747), [#39148](https://github.com/PaddlePaddle/Paddle/pull/39148), [#39416](https://github.com/PaddlePaddle/Paddle/pull/39416))

- 新增优化器相关 API([#40710](https://github.com/PaddlePaddle/Paddle/pull/40710))
  
  - `paddle.incubate.optimizer.functional.minimize_bfgs`，增加二阶优化器 BFGS。
  
  - `paddle.incubate.optimizer.functional.minimize_lbfgs`，增加二阶优化器 L-BFGS。

- 新增 `paddle.incubate.multiprocessing`模块，支持 Tensor（CPU/GPU）在 python 进程间传输。([#37302](https://github.com/PaddlePaddle/Paddle/pull/37302), [#41339](https://github.com/PaddlePaddle/Paddle/pull/41339)) 

#### IR(Intermediate Representation)

- 动态图转静态图
  
  - 变量类型 StaticAnalysis 模块新增支持类似 `a, b = paddle.shape(x)` 的类型标记。([#39245](https://github.com/PaddlePaddle/Paddle/pull/39245)) 
  
  - 新增支持 `InputSpec.name` 作为 Program 缓存 hash key 的计算字段。([#38273](https://github.com/PaddlePaddle/Paddle/pull/38273)) 
  
  - 新增支持 `dict['key'] = x.shape` 语法。([#40611](https://github.com/PaddlePaddle/Paddle/pull/40611)) 
  
  - 新增支持 Pure FP16 训练。([#36944](https://github.com/PaddlePaddle/Paddle/pull/36944)) 
  
  - 新增支持 `for i in [x,y,z]` 语法。([#37259](https://github.com/PaddlePaddle/Paddle/pull/37259)) 
  
  - 新增支持 python3 的 type hint 语法。([#36544](https://github.com/PaddlePaddle/Paddle/pull/36544)) 

- Pass开发
  
  - 新增基于 NVIDIA cuBlasLt Epilogue 的 FC + [relu|gelu] 的前向与反向融合。([#39437](https://github.com/PaddlePaddle/Paddle/pull/39437)）

- Kernel Primitive API
  
  - 新增 GPU 平台 KP 算子，包括 cast、scale、clip、bce_loss、abs_grad、reduce_sum_grad、reduce_mean_grad、clip、bce_loss、full、full_like、distribution、 random、masked_select_kernel、where_index、masked_select_grad、dropout、sigmoid、where、abs_grad。 ([#36203](https://github.com/PaddlePaddle/Paddle/pull/36203), [#36423](https://github.com/PaddlePaddle/Paddle/pull/36423), [#39390](https://github.com/PaddlePaddle/Paddle/pull/39390), [#39734](https://github.com/PaddlePaddle/Paddle/pull/39734), [#38500](https://github.com/PaddlePaddle/Paddle/pull/38500), [#38959](https://github.com/PaddlePaddle/Paddle/pull/38959), [#39197](https://github.com/PaddlePaddle/Paddle/pull/39197/), [#39563](https://github.com/PaddlePaddle/Paddle/pull/39563), [#39666](https://github.com/PaddlePaddle/Paddle/pull/39666), [#40517](https://github.com/PaddlePaddle/Paddle/pull/40517), [#40617](https://github.com/PaddlePaddle/Paddle/pull/40617), [#40766](https://github.com/PaddlePaddle/Paddle/pull/40766), [#39898](https://github.com/PaddlePaddle/Paddle/pull/39898), [#39609](https://github.com/PaddlePaddle/Paddle/pull/39609)) 
  
  - 新增支持 XPU2 源码编译模式。([#37254](https://github.com/PaddlePaddle/Paddle/pull/37254), [#40397](https://github.com/PaddlePaddle/Paddle/pull/40397), [#38455](https://github.com/PaddlePaddle/Paddle/pull/38455)) 
  
  - 新增支持 KP 算子在 XPU2 和 GPU 中复用，包括 reduce、broadcast、elementwise_add、`exp、log、relu、sigmoid、leaky_relu、softplus、hard_swish、reciprocal`。([#36904](https://github.com/PaddlePaddle/Paddle/pull/36904), [#37226](https://github.com/PaddlePaddle/Paddle/pull/37226), [#38918](https://github.com/PaddlePaddle/Paddle/pull/38918), [#40560](https://github.com/PaddlePaddle/Paddle/pull/40560/), [#39787](https://github.com/PaddlePaddle/Paddle/pull/39787), [#39917](https://github.com/PaddlePaddle/Paddle/pull/39917), [#40002](https://github.com/PaddlePaddle/Paddle/pull/40002), [#40364](https://github.com/PaddlePaddle/Paddle/pull/40364))
  
  - 新增 XPU2 平台 KP 算子单测，包括 `brelu、ceil、celu、elu、floor、hard_shrink、hard_sigmoid、log1p、logsigmoid、relu6、silu、soft_relu、softsign、sqrt、square、swish、thresholded_relu、softshrink`。([#40448](https://github.com/PaddlePaddle/Paddle/pull/40448), [#40524](https://github.com/PaddlePaddle/Paddle/pull/40524)) 
  
  - 新增 XPU2 KP 模型支持，包括 resnet50、deepfm、wide_deep、yolov3-darknet53、det_mv3_db、bert、transformer、mobilenet_v3、GPT2。

#### 混合精度训练

- 从混合精度训练 `paddle.amp.GradScaler` 的 `minimize` 中拆分出 `paddle.amp.Gradscaler.unscale_` 方法，提供恢复 loss 的独立接口。([#35825](https://github.com/PaddlePaddle/Paddle/pull/35825)) 

- 为 `paddle.nn.ClipByGlobalNorm` 动态图模式添加 FP16 支持，为clip op 添加 FP16 Kernel，使`clip`相关操作支持 FP16。([#36198](https://github.com/PaddlePaddle/Paddle/pull/36198), [#36577](https://github.com/PaddlePaddle/Paddle/pull/36577))

- 支持 `paddle.amp.decorate` 传入的`optimizer`参数为 None。([#37541](https://github.com/PaddlePaddle/Paddle/pull/37541)) 

- 为 merged_momentum op 添加支持输入多学习率、支持 use_nesterov 策略的计算、支持 regularization 计算。([#37527](https://github.com/PaddlePaddle/Paddle/pull/37527))

- 为`paddle.optimizer.Momentum`优化器添加 multi_tensor 策略、为`Optimzizer`类的`clear_grad`添加`set_to_zero`分支。([#37564](https://github.com/PaddlePaddle/Paddle/pull/37564)) 

- 为`paddle.optimizer.Adam`优化器添加 multi_tensor 策略。([#38010](https://github.com/PaddlePaddle/Paddle/pull/38010)) 

- 为`paddle.optimizer.SGD`优化器添加 multi_precision 策略。([#38231](https://github.com/PaddlePaddle/Paddle/pull/38231)) 

- 为优化器 `state_dict` 方法添加存储 `master weight` 参数。([#39121](https://github.com/PaddlePaddle/Paddle/pull/39121)) 

- 添加支持 op CUDA bfloat16 混合精度训练，支持 O1、O2 模式，通过 `paddle.amp.auto_cast`可开启上述训练模式。([#39029](https://github.com/PaddlePaddle/Paddle/pull/39029), [#39815](https://github.com/PaddlePaddle/Paddle/pull/39815))

- 为如下 ops 添加 bfloat16 CUDA Kernel：matmul、concat、split、dropout、reshape、slice、squeeze、stack、transpose、unbind、elementwize_max、elementwize_add、elementwize_mul、elementwize_sub、scale、sum、layer_norm、p_norm、reduce_sum、softmax、log_softmax、sigmoid、sqrt、softplus、square、gaussian_random、fill_constant、fill_any_like。([#39485](https://github.com/PaddlePaddle/Paddle/pull/39485), [#39380](https://github.com/PaddlePaddle/Paddle/pull/39380), [#39395](https://github.com/PaddlePaddle/Paddle/pull/39380), [#39402](https://github.com/PaddlePaddle/Paddle/pull/39402), [#39457](https://github.com/PaddlePaddle/Paddle/pull/39457), [#39461](https://github.com/PaddlePaddle/Paddle/pull/39461), [#39602](https://github.com/PaddlePaddle/Paddle/pull/39602), [#39716](https://github.com/PaddlePaddle/Paddle/pull/39716), [#39683](https://github.com/PaddlePaddle/Paddle/pull/39683), [#39843](https://github.com/PaddlePaddle/Paddle/pull/39843), [#39999](https://github.com/PaddlePaddle/Paddle/pull/39999), [#40004](https://github.com/PaddlePaddle/Paddle/pull/40004), [#40027](https://github.com/PaddlePaddle/Paddle/pull/40027)) 

- 为如下 ops 添加 bfloat16 CPU Kernel：dropout、reshape、slice、squeeze、unsqueeze、stack、transpose、unbind、elementwize_max、elementwise_mul、elementwise_sub、gather。 ([#39380](https://github.com/PaddlePaddle/Paddle/pull/39380), [#39395](https://github.com/PaddlePaddle/Paddle/pull/39380), [#39402](https://github.com/PaddlePaddle/Paddle/pull/39402), [#39457](https://github.com/PaddlePaddle/Paddle/pull/39457), [#39461](https://github.com/PaddlePaddle/Paddle/pull/39461), [#39602](https://github.com/PaddlePaddle/Paddle/pull/39602), [#39716](https://github.com/PaddlePaddle/Paddle/pull/39716), [#39683](https://github.com/PaddlePaddle/Paddle/pull/39683)) 

- 支持打印 bfloat16 类型的 Tensor。([#39375](https://github.com/PaddlePaddle/Paddle/pull/39375), [#39370](https://github.com/PaddlePaddle/Paddle/pull/39370))

- 为`p_norm`、`elementwise_max` 、`fill_constant_batch_size_like``scatter`增加 FP16 计算支持。([#35888](https://github.com/PaddlePaddle/Paddle/pull/35888), [#39907](https://github.com/PaddlePaddle/Paddle/pull/39907), [#38136](https://github.com/PaddlePaddle/Paddle/pull/38136), [#38499](https://github.com/PaddlePaddle/Paddle/pull/38499))

- 为如下 ops 增加 int16_t 支持：cumsum、less_than、less_equal、greater_than、greater_equal、equal、not_equal、fill_any_like、grather_nd、reduce_sum、where_index、reshape、unsqueeze。([#39636](https://github.com/PaddlePaddle/Paddle/pull/39636)) 

- 为 cross_entropy op 增加 int16_t label 类型的支持。([#39409](https://github.com/PaddlePaddle/Paddle/pull/39409)) 

- 为 embedding op 增加 int16_t id 类型的支持。([#39381](https://github.com/PaddlePaddle/Paddle/pull/39381))

- 为 reduce_mean op 增加 FP16 类型的支持。([#38289](https://github.com/PaddlePaddle/Paddle/pull/38289))

- 为 elementwise_min op 增加 FP16 类型的支持。([#38123](https://github.com/PaddlePaddle/Paddle/pull/38123))

- 更新 bfloat16 AMP oneDNN 默认支持列表。([#39304](https://github.com/PaddlePaddle/Paddle/pull/39304)) 

#### 飞桨高可复用算子库 PHI

针对飞桨框架原算子库存在的算子接口不清晰、算子复用成本较高、调用性能不够快的问题，我们重构了飞桨框架的算子库，设计了灵活、高效的函数式算子库 PHI，可以通过对函数式算子接口组合调用的方式实现新算子。新算子库提供了 200 余个跟 python 开发接口保持一致的 C++ 运算类 API，以及近500个可供组合调用的前、反向函数式算子内核 Kernel，可大幅降低框架原生算子和自定义算子的开发成本。新算子库支持Primitive API方式开发算子内核，可支持不同硬件（比如GPU和XPU）的算子内核复用。新算子库支持以插件方式接入硬件（比如NPU）的加速库，实现低成本复用硬件加速库。主要可分为以下几部分工作：

- **算子库基础架构、核心组件与机制实现**：合理规划新算子库的目录结构，设计实现了新算子库的公共基础数据结构、新的函数式 InferMeta 和 Kernel 开发范式以及相应的注册和管理组件，并且支持 Kernel 文件的自动化编译对象生成及编译依赖关系生成，使开发者仅需关注函数式 Kernel 的实现，开发范式简洁清晰。([#34425](https://github.com/PaddlePaddle/Paddle/pull/34425), [#37107](https://github.com/PaddlePaddle/Paddle/pull/37107), [#36946](https://github.com/PaddlePaddle/Paddle/pull/36946), [#36948](https://github.com/PaddlePaddle/Paddle/pull/36948), [#37876](https://github.com/PaddlePaddle/Paddle/pull/37876), [#37916](https://github.com/PaddlePaddle/Paddle/pull/37916), [#37977](https://github.com/PaddlePaddle/Paddle/pull/37977), [38078](https://github.com/PaddlePaddle/Paddle/pull/38078), [#38861](https://github.com/PaddlePaddle/Paddle/pull/38861), [#39123](https://github.com/PaddlePaddle/Paddle/pull/39123), [#39131](https://github.com/PaddlePaddle/Paddle/pull/39131), [#39748](https://github.com/PaddlePaddle/Paddle/pull/39748), [#39790](https://github.com/PaddlePaddle/Paddle/pull/39790), [#39941](https://github.com/PaddlePaddle/Paddle/pull/39941), [#40239](https://github.com/PaddlePaddle/Paddle/pull/40239), [#40635](https://github.com/PaddlePaddle/Paddle/pull/40635), [#41091](https://github.com/PaddlePaddle/Paddle/pull/41091), [#37409](https://github.com/PaddlePaddle/Paddle/pull/37409), [#37942](https://github.com/PaddlePaddle/Paddle/pull/37942), [#39002](https://github.com/PaddlePaddle/Paddle/pull/39002), [#38109](https://github.com/PaddlePaddle/Paddle/pull/38109), [#37881](https://github.com/PaddlePaddle/Paddle/pull/37881), [#37517](https://github.com/PaddlePaddle/Paddle/pull/37517), [#39870](https://github.com/PaddlePaddle/Paddle/pull/39870), [#40975](https://github.com/PaddlePaddle/Paddle/pull/40975), [#39475](https://github.com/PaddlePaddle/Paddle/pull/39475), [#37304](https://github.com/PaddlePaddle/Paddle/pull/37304), #36910, #37120, #37146, #37215, #37255, #37369, #38258, #38257, #38355, #38853, #38937, #38977, #38946, #39085, #39153, #39228, #38301, #38275, #38506, #38607, #38473, #38632, #38811, #38880, #38996, #38914, #39101)

- **算子库C++ API体系建设**：设计实现了基于 yaml 配置文件的算子定义范式、自动生成了200余个C++运算类 API，供内外部开发者复用，降低了基础运算的重复开发成本。([#37668](https://github.com/PaddlePaddle/Paddle/pull/37668), [#36938](https://github.com/PaddlePaddle/Paddle/pull/36938), [#38172](https://github.com/PaddlePaddle/Paddle/pull/38172), [#38182](https://github.com/PaddlePaddle/Paddle/pull/38182), [#38311](https://github.com/PaddlePaddle/Paddle/pull/38311), [#38438](https://github.com/PaddlePaddle/Paddle/pull/38438), [#39057](https://github.com/PaddlePaddle/Paddle/pull/39057), [#39229](https://github.com/PaddlePaddle/Paddle/pull/39229), [#39281](https://github.com/PaddlePaddle/Paddle/pull/39281), [#39263](https://github.com/PaddlePaddle/Paddle/pull/39263), [#39408](https://github.com/PaddlePaddle/Paddle/pull/39408), [#39436](https://github.com/PaddlePaddle/Paddle/pull/39436), [#39482](https://github.com/PaddlePaddle/Paddle/pull/39482), [#39497](https://github.com/PaddlePaddle/Paddle/pull/39497), [#39651](https://github.com/PaddlePaddle/Paddle/pull/39651), [#39521](https://github.com/PaddlePaddle/Paddle/pull/39521), [#39760](https://github.com/PaddlePaddle/Paddle/pull/39760), [#40060](https://github.com/PaddlePaddle/Paddle/pull/40060), [#40196](https://github.com/PaddlePaddle/Paddle/pull/40196), [#40218](https://github.com/PaddlePaddle/Paddle/pull/40218), [#40640](https://github.com/PaddlePaddle/Paddle/pull/40640), [#40732](https://github.com/PaddlePaddle/Paddle/pull/40732), [#40729](https://github.com/PaddlePaddle/Paddle/pull/40729), [#40840](https://github.com/PaddlePaddle/Paddle/pull/40840), [#40867](https://github.com/PaddlePaddle/Paddle/pull/40867), [#41025](https://github.com/PaddlePaddle/Paddle/pull/41025), [#41368](https://github.com/PaddlePaddle/Paddle/pull/41368))

- **算子库兼容各执行体系**：实现新的 InferMeta 及 Kernel 接入原动静态图执行体系、支持原 OpKernel 注册安全移除并迁移为新的 Kernel 形式。([#34425](https://github.com/PaddlePaddle/Paddle/pull/34425), [#38825](https://github.com/PaddlePaddle/Paddle/pull/38825), [#38837](https://github.com/PaddlePaddle/Paddle/pull/38837), [#38842](https://github.com/PaddlePaddle/Paddle/pull/38842), [#38976](https://github.com/PaddlePaddle/Paddle/pull/38976), [#39134](https://github.com/PaddlePaddle/Paddle/pull/39134), [#39140](https://github.com/PaddlePaddle/Paddle/pull/39140), [#39135](https://github.com/PaddlePaddle/Paddle/pull/39135), [#39252](https://github.com/PaddlePaddle/Paddle/pull/39252), [#39222](https://github.com/PaddlePaddle/Paddle/pull/39222), [#39351](https://github.com/PaddlePaddle/Paddle/pull/39351))

- **算子库底层数据结构及工具函数与框架解耦**：解除 Phi 在核心数据结构上对 框架的依赖，为后续 Phi 独立编译奠定基础，支持 infrt、自定义 Kernel 等一系列基于 Phi 的建设工作 ([#38583](https://github.com/PaddlePaddle/Paddle/pull/38583), [#39188](https://github.com/PaddlePaddle/Paddle/pull/39188), [#39560](https://github.com/PaddlePaddle/Paddle/pull/39560), [#39931](https://github.com/PaddlePaddle/Paddle/pull/39931), [#39169](https://github.com/PaddlePaddle/Paddle/pull/39169), [#38951](https://github.com/PaddlePaddle/Paddle/pull/38951), [#38898](https://github.com/PaddlePaddle/Paddle/pull/38898), [#38873](https://github.com/PaddlePaddle/Paddle/pull/38873), [#38696](https://github.com/PaddlePaddle/Paddle/pull/38696), [#38651](https://github.com/PaddlePaddle/Paddle/pull/38651), [#39359](https://github.com/PaddlePaddle/Paddle/pull/39359), [#39305](https://github.com/PaddlePaddle/Paddle/pull/39305), [#39234](https://github.com/PaddlePaddle/Paddle/pull/39234), [#39098](https://github.com/PaddlePaddle/Paddle/pull/39098), [#39120](https://github.com/PaddlePaddle/Paddle/pull/39120), [#38979](https://github.com/PaddlePaddle/Paddle/pull/38979), [#38899](https://github.com/PaddlePaddle/Paddle/pull/38899), [#38844](https://github.com/PaddlePaddle/Paddle/pull/38844), [#39714](https://github.com/PaddlePaddle/Paddle/pull/39714), [#39729](https://github.com/PaddlePaddle/Paddle/pull/39729), [#39889](https://github.com/PaddlePaddle/Paddle/pull/39889), [#39587](https://github.com/PaddlePaddle/Paddle/pull/39587), [#39558](https://github.com/PaddlePaddle/Paddle/pull/39558), [#39514](https://github.com/PaddlePaddle/Paddle/pull/39514), [#39502](https://github.com/PaddlePaddle/Paddle/pull/39502), [#39300](https://github.com/PaddlePaddle/Paddle/pull/39300), [#39246](https://github.com/PaddlePaddle/Paddle/pull/39246), [#39124](https://github.com/PaddlePaddle/Paddle/pull/39124))

- **自定义算子机制与 Phi 整合并完善**：支持在自定义算子编写时调用 Phi 自动生成的200余个C++运算类 API，降低自定义算子开发成本，并进行一系列问题修复。([#37122](https://github.com/PaddlePaddle/Paddle/pull/37122), [#37276](https://github.com/PaddlePaddle/Paddle/pull/37276), [#37281](https://github.com/PaddlePaddle/Paddle/pull/37281), [#37262](https://github.com/PaddlePaddle/Paddle/pull/37281), [#37415](https://github.com/PaddlePaddle/Paddle/pull/37415), [#37423](https://github.com/PaddlePaddle/Paddle/pull/37423), [#37583](https://github.com/PaddlePaddle/Paddle/pull/37683), [#38776](https://github.com/PaddlePaddle/Paddle/pull/38776), [#39353](https://github.com/PaddlePaddle/Paddle/pull/39353), [#41072](https://github.com/PaddlePaddle/Paddle/pull/41072))

- **算子规模化迁移改写**：迁移了约250个高频算子的前、反向算子内核 Kernel 至新算子库，改写为函数式，支持在 C++端通过调用多个基础 Kernel 函数封装，快速组合实现高性能算子；同时，添加相应的 yaml 算子定义，并接入新动态图执行体系，提升 python API 调度性能。迁移改写的算子包括：
  
  - sqrt （[#40727](https://github.com/PaddlePaddle/Paddle/pull/40727)）
  
  - square（[#40727](https://github.com/PaddlePaddle/Paddle/pull/40727)）
  
  - sin ([#40175](https://github.com/PaddlePaddle/Paddle/pull/40175))
  
  - sinh ([#40175](https://github.com/PaddlePaddle/Paddle/pull/40175))
  
  - elementwise_fmax（[#40140](https://github.com/PaddlePaddle/Paddle/pull/40140)）
  
  - elementwise_fmin（[#40140](https://github.com/PaddlePaddle/Paddle/pull/40140)）
  
  - pool2d（[#40208](https://github.com/PaddlePaddle/Paddle/pull/40208), [#41053](https://github.com/PaddlePaddle/Paddle/pull/41053)）
  
  - max_pool2d_with_index（[#40208](https://github.com/PaddlePaddle/Paddle/pull/40208), [#41053](https://github.com/PaddlePaddle/Paddle/pull/41053)）
  
  - pool3d（[#40208](https://github.com/PaddlePaddle/Paddle/pull/40208), [#41053](https://github.com/PaddlePaddle/Paddle/pull/41053)）
  
  - max_pool3d_with_index（[#40208](https://github.com/PaddlePaddle/Paddle/pull/40208), [#41053](https://github.com/PaddlePaddle/Paddle/pull/41053)）
  
  - fill_constant ([#36930](https://github.com/PaddlePaddle/Paddle/pull/36930), [#39465](https://github.com/PaddlePaddle/Paddle/pull/39465))
  
  - p_norm ([#40819](https://github.com/PaddlePaddle/Paddle/pull/40819))
  
  - fill_constant_batch_size_like ([#40784](https://github.com/PaddlePaddle/Paddle/pull/40784))
  
  - conv2d（[#39354](https://github.com/PaddlePaddle/Paddle/pull/39354)）
  
  - conv2d_transpose（[#40675](https://github.com/PaddlePaddle/Paddle/pull/40675), [#41053](https://github.com/PaddlePaddle/Paddle/pull/41053)）
  
  - conv3d（[#39354](https://github.com/PaddlePaddle/Paddle/pull/39354)）
  
  - conv3d_transpose（[#40675](https://github.com/PaddlePaddle/Paddle/pull/40675), [#41053](https://github.com/PaddlePaddle/Paddle/pull/41053)）
  
  - mish（[#40727](https://github.com/PaddlePaddle/Paddle/pull/40727)）
  
  - gather_nd ([#40090](https://github.com/PaddlePaddle/Paddle/pull/40090), [#40043](https://github.com/PaddlePaddle/Paddle/pull/40043))
  
  - gather ([#40500](https://github.com/PaddlePaddle/Paddle/pull/40500))
  
  - scatter ([#40090](https://github.com/PaddlePaddle/Paddle/pull/40090), [#40043](https://github.com/PaddlePaddle/Paddle/pull/40043))
  
  - scatter_nd_add ([#40090](https://github.com/PaddlePaddle/Paddle/pull/40090), [#40043](https://github.com/PaddlePaddle/Paddle/pull/40043))
  
  - sgd（[40045](https://github.com/PaddlePaddle/Paddle/pull/40045)）
  
  - momentum ([#41319](https://github.com/PaddlePaddle/Paddle/pull/41319))
  
  - rmsprop（[#40994](https://github.com/PaddlePaddle/Paddle/pull/40994)）
  
  - index_sample（[#38130](https://github.com/PaddlePaddle/Paddle/pull/38130), [#38459](https://github.com/PaddlePaddle/Paddle/pull/38459),[#39905](https://github.com/PaddlePaddle/Paddle/pull/39905)）
  
  - adam ([#40351](https://github.com/PaddlePaddle/Paddle/pull/40351))
  
  - layer_norm（[#40193](https://github.com/PaddlePaddle/Paddle/pull/40193)）
  
  - adagrad（[#40994](https://github.com/PaddlePaddle/Paddle/pull/40994/)）
  
  - adamax ([#40173](https://github.com/PaddlePaddle/Paddle/pull/40173))
  
  - adadelta ([#40173](https://github.com/PaddlePaddle/Paddle/pull/40173))
  
  - clip（[#40602](https://github.com/PaddlePaddle/Paddle/pull/40602), [#41661](https://github.com/PaddlePaddle/Paddle/pull/41661), [#41675](https://github.com/PaddlePaddle/Paddle/pull/41675)）
  
  - ceil ([#40913](https://github.com/PaddlePaddle/Paddle/pull/40913))
  
  - cos ([#40175](https://github.com/PaddlePaddle/Paddle/pull/40175))
  
  - atan ([#40175](https://github.com/PaddlePaddle/Paddle/pull/40175))
  
  - cosh ([#40175](https://github.com/PaddlePaddle/Paddle/pull/40175))
  
  - erf（[#40388](https://github.com/PaddlePaddle/Paddle/pull/40388)）
  
  - asin ([#40175](https://github.com/PaddlePaddle/Paddle/pull/40175))
  
  - acos ([#40175](https://github.com/PaddlePaddle/Paddle/pull/40175))
  
  - scale ([#39278](https://github.com/PaddlePaddle/Paddle/pull/39278))
  
  - elementwise_pow ([#40993](https://github.com/PaddlePaddle/Paddle/pull/40993))
  
  - elementwise_sub ([#39225](https://github.com/PaddlePaddle/Paddle/pull/39225), [#37260](https://github.com/PaddlePaddle/Paddle/pull/37260))
  
  - round ([#40913](https://github.com/PaddlePaddle/Paddle/pull/40913))
  
  - floor ([#40913](https://github.com/PaddlePaddle/Paddle/pull/40913))
  
  - pow ([#40913](https://github.com/PaddlePaddle/Paddle/pull/40913))
  
  - elementwise_floordiv ([#40993](https://github.com/PaddlePaddle/Paddle/pull/40993))
  
  - reciprocal（[#40727](https://github.com/PaddlePaddle/Paddle/pull/40727)）
  
  - log1p ([#40785](https://github.com/PaddlePaddle/Paddle/pull/40785))
  
  - allclose ([#40469](https://github.com/PaddlePaddle/Paddle/pull/40469))
  
  - mul ([#40833](https://github.com/PaddlePaddle/Paddle/pull/40833))
  
  - elementwise_max ([#40590](https://github.com/PaddlePaddle/Paddle/pull/40590))
  
  - elementwise_min ([#40590](https://github.com/PaddlePaddle/Paddle/pull/40590))
  
  - elementwise_mod ([#40590](https://github.com/PaddlePaddle/Paddle/pull/40590))
  
  - elementwise_add ([#39048](https://github.com/PaddlePaddle/Paddle/pull/39048), [#37043](https://github.com/PaddlePaddle/Paddle/pull/37043))
  
  - matmul_v2 ([#36844](https://github.com/PaddlePaddle/Paddle/pull/36844), [#38713](https://github.com/PaddlePaddle/Paddle/pull/38713))
  
  - elementwise_mul ([#41042](https://github.com/PaddlePaddle/Paddle/pull/41042), [#40252](https://github.com/PaddlePaddle/Paddle/pull/40252), [#37471](https://github.com/PaddlePaddle/Paddle/pull/37471))
  
  - elementwise_div ([#40172](https://github.com/PaddlePaddle/Paddle/pull/40172), [#40039](https://github.com/PaddlePaddle/Paddle/pull/40039), [#37418](https://github.com/PaddlePaddle/Paddle/pull/37418))
  
  - SelectedRows ([#39037](https://github.com/PaddlePaddle/Paddle/pull/39037), [#39087](https://github.com/PaddlePaddle/Paddle/pull/39087), [#39128](https://github.com/PaddlePaddle/Paddle/pull/39128), [#39162](https://github.com/PaddlePaddle/Paddle/pull/39162), [#39236](https://github.com/PaddlePaddle/Paddle/pull/39236)) 
  
  - fill_any_like ([#39807](https://github.com/PaddlePaddle/Paddle/pull/39807))
  
  - dot（[#38359](https://github.com/PaddlePaddle/Paddle/pull/38359)）
  
  - sum ([#40873](https://github.com/PaddlePaddle/Paddle/pull/40873))
  
  - cumsum ([#39976](https://github.com/PaddlePaddle/Paddle/pull/39976), [#40200](https://github.com/PaddlePaddle/Paddle/pull/40200))
  
  - diag_v2 ([#39914](https://github.com/PaddlePaddle/Paddle/pull/39914))
  
  - auc ([#39976](https://github.com/PaddlePaddle/Paddle/pull/39976), [#40200](https://github.com/PaddlePaddle/Paddle/pull/40200))
  
  - log_loss ([#39976](https://github.com/PaddlePaddle/Paddle/pull/39976), [#40200](https://github.com/PaddlePaddle/Paddle/pull/40200))
  
  - one_hot_v2（[39876](https://github.com/PaddlePaddle/Paddle/pull/39876)）
  
  - sigmoid_cross_entropy_with_logits ([#39976](https://github.com/PaddlePaddle/Paddle/pull/39976), [#40200](https://github.com/PaddlePaddle/Paddle/pull/40200))
  
  - bce_loss ([#39868](https://github.com/PaddlePaddle/Paddle/pull/39868))
  
  - argsort ([#40151](https://github.com/PaddlePaddle/Paddle/pull/40151))
  
  - arg_max ([#40222](https://github.com/PaddlePaddle/Paddle/pull/40222))
  
  - arg_min ([#40222](https://github.com/PaddlePaddle/Paddle/pull/40222))
  
  - segment_pool ([#40099](https://github.com/PaddlePaddle/Paddle/pull/40099))
  
  - frobenius_norm（[#40707](https://github.com/PaddlePaddle/Paddle/pull/40707), [#41053](https://github.com/PaddlePaddle/Paddle/pull/41053)）
  
  - dist ([#40178](https://github.com/PaddlePaddle/Paddle/pull/40178))
  
  - isnan_v2 ([#40076](https://github.com/PaddlePaddle/Paddle/pull/40076))
  
  - logical_and ([#39942](https://github.com/PaddlePaddle/Paddle/pull/39942))
  
  - logical_not ([#39942](https://github.com/PaddlePaddle/Paddle/pull/39942))
  
  - isfinite_v2 ([#40076](https://github.com/PaddlePaddle/Paddle/pull/40076))
  
  - logical_or ([#39942](https://github.com/PaddlePaddle/Paddle/pull/39942))
  
  - isinf_v2 ([#40076](https://github.com/PaddlePaddle/Paddle/pull/40076))
  
  - is_empty ([#39919](https://github.com/PaddlePaddle/Paddle/pull/39919))
  
  - logical_xor ([#39942](https://github.com/PaddlePaddle/Paddle/pull/39942))
  
  - less_than（[#39970](https://github.com/PaddlePaddle/Paddle/pull/39970)）
  
  - not_equal（[#39970](https://github.com/PaddlePaddle/Paddle/pull/39970)）
  
  - equal（[#39970](https://github.com/PaddlePaddle/Paddle/pull/39970)）
  
  - less_equal（[#39970](https://github.com/PaddlePaddle/Paddle/pull/39970)）
  
  - equal_all（[#39970](https://github.com/PaddlePaddle/Paddle/pull/39970)）
  
  - uniform_random ([#39937](https://github.com/PaddlePaddle/Paddle/pull/39937))
  
  - randint ([#39876](https://github.com/PaddlePaddle/Paddle/pull/39876), [#41375](https://github.com/PaddlePaddle/Paddle/pull/41375))
  
  - randperm ([#41265](https://github.com/PaddlePaddle/Paddle/pull/41265))
  
  - unbind ([#39789](https://github.com/PaddlePaddle/Paddle/pull/39789))
  
  - bernoulli ([#39590](https://github.com/PaddlePaddle/Paddle/pull/39590))
  
  - increment ([#39858](https://github.com/PaddlePaddle/Paddle/pull/39858), [#39913](https://github.com/PaddlePaddle/Paddle/pull/39913))
  
  - multinomial ([#39858](https://github.com/PaddlePaddle/Paddle/pull/39858), [#39913](https://github.com/PaddlePaddle/Paddle/pull/39913))
  
  - addmm ([#39858](https://github.com/PaddlePaddle/Paddle/pull/39858), [#39913](https://github.com/PaddlePaddle/Paddle/pull/39913))
  
  - cholesky ([#39858](https://github.com/PaddlePaddle/Paddle/pull/39858), [#39913](https://github.com/PaddlePaddle/Paddle/pull/39913))
  
  - where ([#39811](https://github.com/PaddlePaddle/Paddle/pull/39811))
  
  - log10 ([#40785](https://github.com/PaddlePaddle/Paddle/pull/40785))
  
  - log2 ([#40785](https://github.com/PaddlePaddle/Paddle/pull/40785))
  
  - expm1（[#40727](https://github.com/PaddlePaddle/Paddle/pull/40727)）
  
  - atan2 ([#39806](https://github.com/PaddlePaddle/Paddle/pull/39806))
  
  - gaussian_random ([#39932](https://github.com/PaddlePaddle/Paddle/pull/39932), [#40122](https://github.com/PaddlePaddle/Paddle/pull/40122), [#40191](https://github.com/PaddlePaddle/Paddle/pull/40191))
  
  - empty ([#38334](https://github.com/PaddlePaddle/Paddle/pull/38334))
  
  - truncated_gaussian_random ([#39971](https://github.com/PaddlePaddle/Paddle/pull/39971), [#40191](https://github.com/PaddlePaddle/Paddle/pull/40191))
  
  - mv ([#39861](https://github.com/PaddlePaddle/Paddle/pull/39861), [#39954](https://github.com/PaddlePaddle/Paddle/pull/39954))
  
  - tan ([#40175](https://github.com/PaddlePaddle/Paddle/pull/40175))
  
  - set_value ([#40195](https://github.com/PaddlePaddle/Paddle/pull/40195), [#40478](https://github.com/PaddlePaddle/Paddle/pull/40478), [#40636](https://github.com/PaddlePaddle/Paddle/pull/40636))
  
  - bitwise_and （[#40031](https://github.com/PaddlePaddle/Paddle/pull/40031)）
  
  - bitwise_not（[#40031](https://github.com/PaddlePaddle/Paddle/pull/40031)）
  
  - bitwise_or（[#40031](https://github.com/PaddlePaddle/Paddle/pull/40031)）
  
  - poisson（[#39814](https://github.com/PaddlePaddle/Paddle/pull/39814)）
  
  - cholesky_solve（[#40387](https://github.com/PaddlePaddle/Paddle/pull/40387)）
  
  - bitwise_xor（[#40031](https://github.com/PaddlePaddle/Paddle/pull/40031)）
  
  - triangular_solve（[#40417](https://github.com/PaddlePaddle/Paddle/pull/40417)）
  
  - sigmoid ([#40626](https://github.com/PaddlePaddle/Paddle/pull/40626))
  
  - atanh ([#40175](https://github.com/PaddlePaddle/Paddle/pull/40175))
  
  - softsign（[#40727](https://github.com/PaddlePaddle/Paddle/pull/40727)）
  
  - thresholded_relu ([#40385](https://github.com/PaddlePaddle/Paddle/pull/40385))
  
  - tanh_shrink ([#40565](https://github.com/PaddlePaddle/Paddle/pull/40565))
  
  - stanh（[#40727](https://github.com/PaddlePaddle/Paddle/pull/40727)）
  
  - reduce_mean ([#37559](https://github.com/PaddlePaddle/Paddle/pull/37559))
  
  - reduce_max（[#40225](https://github.com/PaddlePaddle/Paddle/pull/40225)）
  
  - reduce_min ([#40374](https://github.com/PaddlePaddle/Paddle/pull/40374))
  
  - mean ([#40872](https://github.com/PaddlePaddle/Paddle/pull/40872), [#41319](https://github.com/PaddlePaddle/Paddle/pull/41319))
  
  - reduce_all ([#40374](https://github.com/PaddlePaddle/Paddle/pull/40374))
  
  - reduce_any ([#40374](https://github.com/PaddlePaddle/Paddle/pull/40374))
  
  - logsumexp ([#40790](https://github.com/PaddlePaddle/Paddle/pull/40790))
  
  - softshrink（[#40565](https://github.com/PaddlePaddle/Paddle/pull/40565)）
  
  - range ([#41265](https://github.com/PaddlePaddle/Paddle/pull/41265), [#40581](https://github.com/PaddlePaddle/Paddle/pull/40851))
  
  - stack（[#40581](https://github.com/PaddlePaddle/Paddle/pull/40851)）
  
  - tile ([#40371](https://github.com/PaddlePaddle/Paddle/pull/40371))
  
  - unique（[#40581](https://github.com/PaddlePaddle/Paddle/pull/40851)）
  
  - unstack（[#40581](https://github.com/PaddlePaddle/Paddle/pull/40851)）
  
  - slice（[#40736](https://github.com/PaddlePaddle/Paddle/pull/40736)）
  
  - transpose2（[#39327](https://github.com/PaddlePaddle/Paddle/pull/39327)）
  
  - unsqueeze2（ [#40596](https://github.com/PaddlePaddle/Paddle/pull/40596)）
  
  - squeeze2（ [#40596](https://github.com/PaddlePaddle/Paddle/pull/40596)）
  
  - strided_slice ([#40708](https://github.com/PaddlePaddle/Paddle/pull/40708))
  
  - softmax ([#39547](https://github.com/PaddlePaddle/Paddle/pull/39547))
  
  - leaky_relu ([#40385](https://github.com/PaddlePaddle/Paddle/pull/40385))
  
  - gelu ([#40393](https://github.com/PaddlePaddle/Paddle/pull/40393))
  
  - prelu ([#40393](https://github.com/PaddlePaddle/Paddle/pull/40393))
  
  - log_softmax ([#40393](https://github.com/PaddlePaddle/Paddle/pull/40393))
  
  - elu ([#40565](https://github.com/PaddlePaddle/Paddle/pull/40565))
  
  - logsigmoid ([#40626](https://github.com/PaddlePaddle/Paddle/pull/40626))
  
  - psroi_pool ([#40353](https://github.com/PaddlePaddle/Paddle/pull/40353), [#41173](https://github.com/PaddlePaddle/Paddle/pull/41173))
  
  - kthvalue（[#40575](https://github.com/PaddlePaddle/Paddle/pull/40575)）
  
  - mode ([#40571](https://github.com/PaddlePaddle/Paddle/pull/40571))
  
  - yolo_box（[#40112](https://github.com/PaddlePaddle/Paddle/pull/40112)）
  
  - yolov3_loss ([#40944](https://github.com/PaddlePaddle/Paddle/pull/40944)）
  
  - temporal_shift（[#40727](https://github.com/PaddlePaddle/Paddle/pull/40727)）
  
  - depthwise_conv2d（[#39354](https://github.com/PaddlePaddle/Paddle/pull/39354)）
  
  - pad3d ([#40701](https://github.com/PaddlePaddle/Paddle/pull/40701))
  
  - pad（ [#40012](https://github.com/PaddlePaddle/Paddle/pull/40012)）
  
  - greater_equal（[#39970](https://github.com/PaddlePaddle/Paddle/pull/39970)）
  
  - kldiv_loss ([#39770](https://github.com/PaddlePaddle/Paddle/pull/39770))
  
  - isclose ([#39770](https://github.com/PaddlePaddle/Paddle/pull/39770))
  
  - silu ([#40565](https://github.com/PaddlePaddle/Paddle/pull/40565))
  
  - unfold ([#39778](https://github.com/PaddlePaddle/Paddle/pull/39778))
  
  - batch_norm（[39347](https://github.com/PaddlePaddle/Paddle/pull/39347)）
  
  - norm（[#39324](https://github.com/PaddlePaddle/Paddle/pull/39324)）
  
  - roi_pool ([#40574](https://github.com/PaddlePaddle/Paddle/pull/40574), [#40682](https://github.com/PaddlePaddle/Paddle/pull/40682), [#41173](https://github.com/PaddlePaddle/Paddle/pull/41173))
  
  - roi_align ([#40382](https://github.com/PaddlePaddle/Paddle/pull/40382), [#40556](https://github.com/PaddlePaddle/Paddle/pull/40556), [#41402](https://github.com/PaddlePaddle/Paddle/pull/41402))
  
  - deformable_conv ([#40700](https://github.com/PaddlePaddle/Paddle/pull/40700), [#40794](https://github.com/PaddlePaddle/Paddle/pull/40794), [#41644](https://github.com/PaddlePaddle/Paddle/pull/41644))
  
  - deformable_conv_v1 ([#40794](https://github.com/PaddlePaddle/Paddle/pull/40794), [#41644](https://github.com/PaddlePaddle/Paddle/pull/41644))
  
  - label_smooth ([#39796](https://github.com/PaddlePaddle/Paddle/pull/39796))
  
  - grid_sampler ([#40585](https://github.com/PaddlePaddle/Paddle/pull/40585))
  
  - greater_than（[#39970](https://github.com/PaddlePaddle/Paddle/pull/39970)）
  
  - pixel_shuffle ([#39949](https://github.com/PaddlePaddle/Paddle/pull/39949), [#39712](https://github.com/PaddlePaddle/Paddle/pull/39712))
  
  - nearest_interp_v2 ([#40855](https://github.com/PaddlePaddle/Paddle/pull/40855))
  
  - bilinear_interp_v2 ([#40855](https://github.com/PaddlePaddle/Paddle/pull/40855))
  
  - softmax_with_cross_entropy ([#40832](https://github.com/PaddlePaddle/Paddle/pull/40832))
  
  - rnn ([#41007](https://github.com/PaddlePaddle/Paddle/pull/41007))
  
  - reverse ([#40791](https://github.com/PaddlePaddle/Paddle/pull/40791))
  
  - trace ([#39510](https://github.com/PaddlePaddle/Paddle/pull/39510))
  
  - kron（[#40427](https://github.com/PaddlePaddle/Paddle/pull/40427)）
  
  - accuracy（[#39982](https://github.com/PaddlePaddle/Paddle/pull/39982)）
  
  - gather_tree ([#40082](https://github.com/PaddlePaddle/Paddle/pull/40082), [#39844](https://github.com/PaddlePaddle/Paddle/pull/39844))
  
  - dropout（[#40148](https://github.com/PaddlePaddle/Paddle/pull/40148)）
  
  - bincount ([#39947](https://github.com/PaddlePaddle/Paddle/pull/39947))
  
  - warpctc ([#41389](https://github.com/PaddlePaddle/Paddle/pull/41389), [#40023](https://github.com/PaddlePaddle/Paddle/pull/https://github.com/PaddlePaddle/Paddle/pull/40023))
  
  - multiplex（[#40007](https://github.com/PaddlePaddle/Paddle/pull/40007), [#40102](https://github.com/PaddlePaddle/Paddle/pull/40102)）
  
  - qr（[#40007](https://github.com/PaddlePaddle/Paddle/pull/40007), [#40007](https://github.com/PaddlePaddle/Paddle/pull/40007)）
  
  - assign_value ([#40967](https://github.com/PaddlePaddle/Paddle/pull/40967))
  
  - assign ([#40022](https://github.com/PaddlePaddle/Paddle/pull/40022))
  
  - cast ([#37610](https://github.com/PaddlePaddle/Paddle/pull/37610))
  
  - tril_triu（[#40007](https://github.com/PaddlePaddle/Paddle/pull/40007), [#41053](https://github.com/PaddlePaddle/Paddle/pull/41053)）
  
  - where_index ([#40255](https://github.com/PaddlePaddle/Paddle/pull/40255))
  
  - index_select ([#40260](https://github.com/PaddlePaddle/Paddle/pull/40260), [#41053](https://github.com/PaddlePaddle/Paddle/pull/41053))
  
  - roll ([#40257](https://github.com/PaddlePaddle/Paddle/pull/40257), [#41053](https://github.com/PaddlePaddle/Paddle/pull/41053))
  
  - cumprod (熊昆 [#39770](https://github.com/PaddlePaddle/Paddle/pull/39770))
  
  - shard_index ([#40254](https://github.com/PaddlePaddle/Paddle/pull/40254))
  
  - reshape2 ([#40914](https://github.com/PaddlePaddle/Paddle/pull/40914), [#39631](https://github.com/PaddlePaddle/Paddle/pull/39631), [#38833](https://github.com/PaddlePaddle/Paddle/pull/38833), [#37164](https://github.com/PaddlePaddle/Paddle/pull/37164))
  
  - flip ([#39822](https://github.com/PaddlePaddle/Paddle/pull/39822), [#40974](https://github.com/PaddlePaddle/Paddle/pull/40974))
  
  - eye ([#39712](https://github.com/PaddlePaddle/Paddle/pull/39712), [#40105](https://github.com/PaddlePaddle/Paddle/pull/40105), [#41476](https://github.com/PaddlePaddle/Paddle/pull/41476))
  
  - lookup_table_v2（[#39901](https://github.com/PaddlePaddle/Paddle/pull/39901)）
  
  - searchsorted（[#40520](https://github.com/PaddlePaddle/Paddle/pull/40520), [#41053](https://github.com/PaddlePaddle/Paddle/pull/41053)）
  
  - adamw ([#40351](https://github.com/PaddlePaddle/Paddle/pull/40351))
  
  - tanh ([#40385](https://github.com/PaddlePaddle/Paddle/pull/40385))
  
  - cross ([#39829](https://github.com/PaddlePaddle/Paddle/pull/39829))
  
  - concat ([#38955](https://github.com/PaddlePaddle/Paddle/pull/38955), [#41112](https://github.com/PaddlePaddle/Paddle/pull/41112))
  
  - split ([#39060](https://github.com/PaddlePaddle/Paddle/pull/39060))
  
  - linspace ([#40124](https://github.com/PaddlePaddle/Paddle/pull/40124))
  
  - huber_loss ([#39761](https://github.com/PaddlePaddle/Paddle/pull/39761))
  
  - hierarchical_sigmoid（[#40553](https://github.com/PaddlePaddle/Paddle/pull/40553)）
  
  - nll_loss ([#39936](https://github.com/PaddlePaddle/Paddle/pull/https://github.com/PaddlePaddle/Paddle/pull/39936))
  
  - graph_send_recv ([#40092](https://github.com/PaddlePaddle/Paddle/pull/40092), [#40320](https://github.com/PaddlePaddle/Paddle/pull/40320))
  
  - abs（[#39492](https://github.com/PaddlePaddle/Paddle/pull/39492), [#39762](https://github.com/PaddlePaddle/Paddle/pull/39762)）
  
  - exp（[#40727](https://github.com/PaddlePaddle/Paddle/pull/40727)）
  
  - rsqrt（[#40727](https://github.com/PaddlePaddle/Paddle/pull/40727)）
  
  - viterbi_decode ([#40186](https://github.com/PaddlePaddle/Paddle/pull/40186))
  
  - conj ([#38247](https://github.com/PaddlePaddle/Paddle/pull/38247))
  
  - real ([#39777](https://github.com/PaddlePaddle/Paddle/pull/39777), [#41173](https://github.com/PaddlePaddle/Paddle/pull/41173))
  
  - imag ([#39777](https://github.com/PaddlePaddle/Paddle/pull/39777), [#41173](https://github.com/PaddlePaddle/Paddle/pull/41173))
  
  - take_along_axis ([#39959](https://github.com/PaddlePaddle/Paddle/pull/39959), [#40270](https://github.com/PaddlePaddle/Paddle/pull/40270), [#40974](https://github.com/PaddlePaddle/Paddle/pull/40974))
  
  - put_along_axis ([#39959](https://github.com/PaddlePaddle/Paddle/pull/39959), [#40974](https://github.com/PaddlePaddle/Paddle/pull/40974))
  
  - lgamma ([#39770](https://github.com/PaddlePaddle/Paddle/pull/39770))
  
  - relu ([#40175](https://github.com/PaddlePaddle/Paddle/pull/40175))
  
  - maxout ([#39959](https://github.com/PaddlePaddle/Paddle/pull/39959), [#40974](https://github.com/PaddlePaddle/Paddle/pull/40974))
  
  - log ([#40785](https://github.com/PaddlePaddle/Paddle/pull/40785))
  
  - bilinear_tensor_product（[#39903](https://github.com/PaddlePaddle/Paddle/pull/39903)）
  
  - flatten_contiguous_range ([#38712](https://github.com/PaddlePaddle/Paddle/pull/38712), [#36957](https://github.com/PaddlePaddle/Paddle/pull/36957), [#41345](https://github.com/PaddlePaddle/Paddle/pull/41345))
  
  - matrix_rank ([#40074](https://github.com/PaddlePaddle/Paddle/pull/40074), [#40519](https://github.com/PaddlePaddle/Paddle/pull/40519), [#41466](https://github.com/PaddlePaddle/Paddle/pull/41466))
  
  - logit ([#37844](https://github.com/PaddlePaddle/Paddle/pull/37844))
  
  - lerp ([#40105](https://github.com/PaddlePaddle/Paddle/pull/40105), [#39524](https://github.com/PaddlePaddle/Paddle/pull/39524))
  
  - erfinv ([#39949](https://github.com/PaddlePaddle/Paddle/pull/39949), [#39712](https://github.com/PaddlePaddle/Paddle/pull/39712))
  
  - broadcast_tensors（[#40047](https://github.com/PaddlePaddle/Paddle/pull/40047)）
  
  - gumbel_softmax（[#39873](https://github.com/PaddlePaddle/Paddle/pull/39873)）
  
  - diagonal （[#39575](https://github.com/PaddlePaddle/Paddle/pull/39575)）
  
  - trunc ([#39543](https://github.com/PaddlePaddle/Paddle/pull/39543), [#39772](https://github.com/PaddlePaddle/Paddle/pull/39772))
  
  - multi_dot ([#40038](https://github.com/PaddlePaddle/Paddle/pull/40038))
  
  - matrix_power ([#40231](https://github.com/PaddlePaddle/Paddle/pull/40231))
  
  - digamma（[#39240](https://github.com/PaddlePaddle/Paddle/pull/39240)）
  
  - masked_select（[#39193](https://github.com/PaddlePaddle/Paddle/pull/39193)）
  
  - determinant ([#40539](https://github.com/PaddlePaddle/Paddle/pull/40539))
  
  - eigh ([#40213](https://github.com/PaddlePaddle/Paddle/pull/40213))
  
  - size ([#39949](https://github.com/PaddlePaddle/Paddle/pull/39949), [#39712](https://github.com/PaddlePaddle/Paddle/pull/39712))
  
  - shape ([#40248](https://github.com/PaddlePaddle/Paddle/pull/40248))
  
  - reduce_sum（[#37559](https://github.com/PaddlePaddle/Paddle/pull/37559), [#41295](https://github.com/PaddlePaddle/Paddle/pull/41295)）
  
  - reduce_prod ([#39844](https://github.com/PaddlePaddle/Paddle/pull/39844))
  
  - histogram（[#39496](https://github.com/PaddlePaddle/Paddle/pull/39496)）
  
  - meshgrid ([#41411](https://github.com/PaddlePaddle/Paddle/pull/41411))
  
  - brelu ([#40385](https://github.com/PaddlePaddle/Paddle/pull/40385))
  
  - hard_swish ([#40913](https://github.com/PaddlePaddle/Paddle/pull/40913))
  
  - hard_shrink ([#40565](https://github.com/PaddlePaddle/Paddle/pull/40565))
  
  - selu (熊昆 [#39819](https://github.com/PaddlePaddle/Paddle/pull/39819))
  
  - expand_v2 ([#39471](https://github.com/PaddlePaddle/Paddle/pull/39471))
  
  - top_k_v2（[#40064](https://github.com/PaddlePaddle/Paddle/pull/40064)）
  
  - expand_as_v2（[#40373](https://github.com/PaddlePaddle/Paddle/pull/40373)）
  
  - swish ([#40913](https://github.com/PaddlePaddle/Paddle/pull/40913))
  
  - hard_sigmoid ([#40626](https://github.com/PaddlePaddle/Paddle/pull/40626))

#### 新动态图执行机制

针对飞桨原动态图执行机制的调度性能、二次开发能力差的问题，我们重构了动态图的底层执行机制。通过全新的调用执行方式，配合 Phi 算子库进行高效的运行时执行，对于 Phi 算子库支持的算子，切换到新动态图模式能体验到调度性能有较大幅度的提升。但是由于整体框架执行机制升级的工作量巨大，且该部分工作耦合了大量 Phi 算子库的工作， 因此在这个版本下我们仍未默认使用该执行方式。如果想要试用可以通过设置环境变量 `FLAGS_enable_eager_mode=1` 来切换使用。具体包括如下内容：

- **新动态图执行机制基础架构、核心组件与机制实现**：静态化动态图相关执行代码，将原本的同质化的算子构建变成针对不同 Phi API 的特异化调用从而极大的优化了调度开销。([#36059](https://github.com/PaddlePaddle/Paddle/pull/36059), [#37323](https://github.com/PaddlePaddle/Paddle/pull/37323), [#37556](https://github.com/PaddlePaddle/Paddle/pull/37556), [#37555](https://github.com/PaddlePaddle/Paddle/pull/37555), [#37478](https://github.com/PaddlePaddle/Paddle/pull/37478), [#37458](https://github.com/PaddlePaddle/Paddle/pull/37458), [#37479](https://github.com/PaddlePaddle/Paddle/pull/37479), [#37599](https://github.com/PaddlePaddle/Paddle/pull/37599), [#37659](https://github.com/PaddlePaddle/Paddle/pull/37659), [#37654](https://github.com/PaddlePaddle/Paddle/pull/37654), [#39200](https://github.com/PaddlePaddle/Paddle/pull/39200), [#39309](https://github.com/PaddlePaddle/Paddle/pull/39309), [#39319](https://github.com/PaddlePaddle/Paddle/pull/39319), [#39414](https://github.com/PaddlePaddle/Paddle/pull/39414), [#39504](https://github.com/PaddlePaddle/Paddle/pull/39504), [#39526](https://github.com/PaddlePaddle/Paddle/pull/39526), [#39878](https://github.com/PaddlePaddle/Paddle/pull/39878), [#39963](https://github.com/PaddlePaddle/Paddle/pull/39963))

- **新动态图执行机制子功能开发、适配**：支持了更加灵活，更加完备的动态图子功能例如 hook，pylayer，double_grad, inplace，amp 等等。([#41396](https://github.com/PaddlePaddle/Paddle/pull/41396), [#40400](https://github.com/PaddlePaddle/Paddle/pull/40400), [#40695](https://github.com/PaddlePaddle/Paddle/pull/40695), [#41043](https://github.com/PaddlePaddle/Paddle/pull/41043), [#40915](https://github.com/PaddlePaddle/Paddle/pull/40915), [#41104](https://github.com/PaddlePaddle/Paddle/pull/41104), [#41350](https://github.com/PaddlePaddle/Paddle/pull/41350), [#41209](https://github.com/PaddlePaddle/Paddle/pull/41209), [#40830](https://github.com/PaddlePaddle/Paddle/pull/40830), [#40891](https://github.com/PaddlePaddle/Paddle/pull/40891), [#36814](https://github.com/PaddlePaddle/Paddle/pull/36814), [#37377](https://github.com/PaddlePaddle/Paddle/pull/37377), [#37193](https://github.com/PaddlePaddle/Paddle/pull/37193), [#36965](https://github.com/PaddlePaddle/Paddle/pull/36965), [#37810](https://github.com/PaddlePaddle/Paddle/pull/37810), [#36837](https://github.com/PaddlePaddle/Paddle/pull/36837), [#38488](https://github.com/PaddlePaddle/Paddle/pull/38488), [#39282](https://github.com/PaddlePaddle/Paddle/pull/39282), [#39449](https://github.com/PaddlePaddle/Paddle/pull/39449), [#39531](https://github.com/PaddlePaddle/Paddle/pull/39531), [#39638](https://github.com/PaddlePaddle/Paddle/pull/39638), [#39674](https://github.com/PaddlePaddle/Paddle/pull/39674), [#39893](https://github.com/PaddlePaddle/Paddle/pull/39893), [#40170](https://github.com/PaddlePaddle/Paddle/pull/40170), [#40693](https://github.com/PaddlePaddle/Paddle/pull/40693), [#40937](https://github.com/PaddlePaddle/Paddle/pull/40937), [#41016](https://github.com/PaddlePaddle/Paddle/pull/41016), [#41051](https://github.com/PaddlePaddle/Paddle/pull/41051), [#41121](https://github.com/PaddlePaddle/Paddle/pull/41121), [#41198](https://github.com/PaddlePaddle/Paddle/pull/41198), [#41287](https://github.com/PaddlePaddle/Paddle/pull/41287), [#41380](https://github.com/PaddlePaddle/Paddle/pull/41380), [#41306](https://github.com/PaddlePaddle/Paddle/pull/41306), [#41387](https://github.com/PaddlePaddle/Paddle/pull/41387), [#40623](https://github.com/PaddlePaddle/Paddle/pull/40623), [#40945](https://github.com/PaddlePaddle/Paddle/pull/40945), [#39282](https://github.com/PaddlePaddle/Paddle/pull/39282), [#39449](https://github.com/PaddlePaddle/Paddle/pull/39449), [#38488](https://github.com/PaddlePaddle/Paddle/pull/38488))

- **新动态图执行的自动代码生成机制**：当我们为了将大量的同质化算子的计算和调度逻辑分化成不同的特异化的调度逻辑时，我们发现这是一个非常庞大的工作，因此我们引入了全新的自动代码生成逻辑来生成代码从而简化动态图的运行时逻辑。同时，为了能够适配之前框架中的各类运行时逻辑，我们也利用了一些复杂的编译手段来运行时的获取信息从而生成更加准确的调度代码。([#37574](https://github.com/PaddlePaddle/Paddle/pull/37574), [#37575](https://github.com/PaddlePaddle/Paddle/pull/37575), [#37639](https://github.com/PaddlePaddle/Paddle/pull/37639), [#37723](https://github.com/PaddlePaddle/Paddle/pull/37723), [#37753](https://github.com/PaddlePaddle/Paddle/pull/37753), [#37812](https://github.com/PaddlePaddle/Paddle/pull/37812), [#37837](https://github.com/PaddlePaddle/Paddle/pull/37837), [#37910](https://github.com/PaddlePaddle/Paddle/pull/37910), [#37943](https://github.com/PaddlePaddle/Paddle/pull/37943), [#37992](https://github.com/PaddlePaddle/Paddle/pull/37992), [#37959](https://github.com/PaddlePaddle/Paddle/pull/37959), [#38017](https://github.com/PaddlePaddle/Paddle/pull/38017), [#37969](https://github.com/PaddlePaddle/Paddle/pull/37969), [#38160](https://github.com/PaddlePaddle/Paddle/pull/38160), [#38085](https://github.com/PaddlePaddle/Paddle/pull/38085), [#38562](https://github.com/PaddlePaddle/Paddle/pull/38562), [#38573](https://github.com/PaddlePaddle/Paddle/pull/38573), [#39192](https://github.com/PaddlePaddle/Paddle/pull/39192), [#39215](https://github.com/PaddlePaddle/Paddle/pull/39215), [#39355](https://github.com/PaddlePaddle/Paddle/pull/39355), [#39358](https://github.com/PaddlePaddle/Paddle/pull/39358), [#39328](https://github.com/PaddlePaddle/Paddle/pull/39328), [#39233](https://github.com/PaddlePaddle/Paddle/pull/39233), [#39628](https://github.com/PaddlePaddle/Paddle/pull/39628), [#39767](https://github.com/PaddlePaddle/Paddle/pull/39767), [#39743](https://github.com/PaddlePaddle/Paddle/pull/39743), [#39897](https://github.com/PaddlePaddle/Paddle/pull/39897), [#39797](https://github.com/PaddlePaddle/Paddle/pull/39797), [#39997](https://github.com/PaddlePaddle/Paddle/pull/39997), [#40058](https://github.com/PaddlePaddle/Paddle/pull/40058), [#40080](https://github.com/PaddlePaddle/Paddle/pull/40080), [#40107](https://github.com/PaddlePaddle/Paddle/pull/40107), [#39962](https://github.com/PaddlePaddle/Paddle/pull/39962), [#40132](https://github.com/PaddlePaddle/Paddle/pull/40132), [#40276](https://github.com/PaddlePaddle/Paddle/pull/40276), [#40266](https://github.com/PaddlePaddle/Paddle/pull/40266), [#40480](https://github.com/PaddlePaddle/Paddle/pull/40480), [#40482](https://github.com/PaddlePaddle/Paddle/pull/40482), [#40368](https://github.com/PaddlePaddle/Paddle/pull/40368), [#40650](https://github.com/PaddlePaddle/Paddle/pull/40650), [#40815](https://github.com/PaddlePaddle/Paddle/pull/40815), [#40907](https://github.com/PaddlePaddle/Paddle/pull/40907), [#40935](https://github.com/PaddlePaddle/Paddle/pull/40935), [#41089](https://github.com/PaddlePaddle/Paddle/pull/41089))

- **新动态图执行机制接入主框架，联合调试**：我们目前利用一些环境变量区分静态图模式和动态图模式（含新动态图和老动态图模式），这些模式下我们已经适配了大部分的动态图的逻辑，但是仍有大量问题正在修复中。([#37638](https://github.com/PaddlePaddle/Paddle/pull/37638), [#37643](https://github.com/PaddlePaddle/Paddle/pull/37643), [#37653](https://github.com/PaddlePaddle/Paddle/pull/37653), [#38314](https://github.com/PaddlePaddle/Paddle/pull/38314), [#38337](https://github.com/PaddlePaddle/Paddle/pull/38337), [#38338](https://github.com/PaddlePaddle/Paddle/pull/38338), [#39164](https://github.com/PaddlePaddle/Paddle/pull/39164), [#39326](https://github.com/PaddlePaddle/Paddle/pull/39326), [#40391](https://github.com/PaddlePaddle/Paddle/pull/40391), [#40201](https://github.com/PaddlePaddle/Paddle/pull/40201), [#40854](https://github.com/PaddlePaddle/Paddle/pull/40854), [#40887](https://github.com/PaddlePaddle/Paddle/pull/40887))

- **更新了动态图下的一些判断逻辑，支持兼容形态下的动态图快速执行路径**：（[#40786](https://github.com/PaddlePaddle/Paddle/pull/40786)）
  
  - 非静态图模式（目前的过渡方案）：`_non_static_mode()`。
  
  - 在动态图模式下且判断在新动态图（推荐的判断逻辑）：`_in_dygrah_mode()`。
  
  - 在动态图模式下且判断在老动态图（不推荐的判断逻辑，在将来的版本中将废弃）：`_in_legacy_dygraph()`。
  
  - 在动态图模式下开启老动态图并关闭新动态图：`_enable_legacy_dygraph()` 或者退出 `_test_eager_guard()`。
  
  - 在动态图模式下开启新动态图并关闭老动态图：`_disable_legacy_dygraph()` 或者 `with _test_eager_guard()`。
  
  - 在静态图或者动态图模式下判断在新动态图：`_in_eager_without_dygraph_check()`。

- **动态图重构后支持 inplace 策略**：输入与输出为同一个 Tensor。
  
  - - 为动态图重构中间态适配 inplace 策略。([#40400](https://github.com/PaddlePaddle/Paddle/pull/40400))
    
    - 为动态图重构最终态适配 inplace 策略。([#40695](https://github.com/PaddlePaddle/Paddle/pull/40695))
    
    - 动态图重构后，为 PyLayer 功能添加 inplace 策略。([#41043](https://github.com/PaddlePaddle/Paddle/pull/41043))
    
    - 动态图重构后，为 Tensor 的 setitem 功能添加 inplace 策略。([#40915](https://github.com/PaddlePaddle/Paddle/pull/40915))
    
    - 动态图重构后添加`_reset_grad_inplace_version`接口，将 Tensor 的梯度的 inplace version 置为0。([#41101](https://github.com/PaddlePaddle/Paddle/pull/41101))
    
    - 反向计算过程中如果不需要前向 Tensor 的值（no need buffer 属性），则不需要对该 Tensor 进行 inplace version 的检测操作。 为 no_need_buffer 的 Tensor 跳过 inplace version 的检查。([#41350](https://github.com/PaddlePaddle/Paddle/pull/41350))
    
    - 统一动态图重构后与重构前对 inplace version 检查的报错信息。([#41209](https://github.com/PaddlePaddle/Paddle/pull/41209))

- **动态图重构后支持 view 策略**：输入与输出 Tensor 共享底层数据。
  
  - - 为动态图重构中间态适配 view 机制。包括`reshape`、`squeeze`、`unsqueeze`、`flatten` API。([#40830](https://github.com/PaddlePaddle/Paddle/pull/40830))
    
    - 为动态图重构最终态适配 view 机制。包括`reshape` API。([#40891](https://github.com/PaddlePaddle/Paddle/pull/40891))

#### 全新静态图执行器

为了解决飞桨原静态图执行器在部分场景下调度性能不够理想，不便于扩展多 stream 等问题，我们实现了全新的性能优越，易于扩展的静态图执行器，充分利用了多 stream、多线程的异步调度能力。新执行器相当于原执行器是兼容升级，目前已在单机单卡场景下默认使用，用户不需要在训练代码中做任何修改即可自动使用。当然，我们也提供了接口来切换回原执行器，用户可以通过设置环境变量 `FLAGS_USE_STANDALONE_EXECUTOR=false` 来切换回原执行器。([#41179](https://github.com/PaddlePaddle/Paddle/pull/41179)) 主要内容如下：

- 基础组件：用于执行器中多线程算子调度的高性能线程池 ([#35470](https://github.com/PaddlePaddle/Paddle/pull/35470), [#35930](https://github.com/PaddlePaddle/Paddle/pull/35930), [#36030](https://github.com/PaddlePaddle/Paddle/pull/36030), [#36480](https://github.com/PaddlePaddle/Paddle/pull/36480), [#36688](https://github.com/PaddlePaddle/Paddle/pull/36688), [#36740](https://github.com/PaddlePaddle/Paddle/pull/36740), [#38335](https://github.com/PaddlePaddle/Paddle/pull/38335), [#40770](https://github.com/PaddlePaddle/Paddle/pull/40770)) 及线程协同组件 ([#38779](https://github.com/PaddlePaddle/Paddle/pull/38779), [#40876](https://github.com/PaddlePaddle/Paddle/pull/40876), [#40912](https://github.com/PaddlePaddle/Paddle/pull/40912))，算子执行后及时地显存回收 ([#37642](https://github.com/PaddlePaddle/Paddle/pull/37642), [#39617](https://github.com/PaddlePaddle/Paddle/pull/39617), [#40859](https://github.com/PaddlePaddle/Paddle/pull/40859))，并行执行器新依赖分析算法 ([#37231](https://github.com/PaddlePaddle/Paddle/pull/37231)) 等。

- 调度逻辑：优化执行器中算子的调度方法，支持多 stream 的多线程异步调度机制，将数据类型、设备、布局等转换改为算子调度以提升性能，支持缓存算子 Kernel 选择，支持选择全新 Phi 算子等。（[#35024](https://github.com/PaddlePaddle/Paddle/pull/35024), [#34922](https://github.com/PaddlePaddle/Paddle/pull/34922), [#35711](https://github.com/PaddlePaddle/Paddle/pull/35711), [#35928](https://github.com/PaddlePaddle/Paddle/pull/35928), [#39458](https://github.com/PaddlePaddle/Paddle/pull/39458)，[#36899](https://github.com/PaddlePaddle/Paddle/pull/36899)）。

- 接口兼容：兼容原执行器的用户接口和功能，如对齐 python 端 Executor.run()、支持 Scope 中管理 Tensor 等，确保用户可以无感知地切换新执行器。 ([#37278](https://github.com/PaddlePaddle/Paddle/pull/37278), [#37379](https://github.com/PaddlePaddle/Paddle/pull/37379), [#37445](https://github.com/PaddlePaddle/Paddle/pull/37445), [#37510](https://github.com/PaddlePaddle/Paddle/pull/37510), [#40955](https://github.com/PaddlePaddle/Paddle/pull/40955), [#41778](https://github.com/PaddlePaddle/Paddle/pull/41178), [#41058](https://github.com/PaddlePaddle/Paddle/pull/41058), [#38584](https://github.com/PaddlePaddle/Paddle/pull/38584), [#37957](https://github.com/PaddlePaddle/Paddle/pull/37957), [#37672](https://github.com/PaddlePaddle/Paddle/pull/37672), [#37474](https://github.com/PaddlePaddle/Paddle/pull/37474), [#37085](https://github.com/PaddlePaddle/Paddle/pull/37085), [#37061](https://github.com/PaddlePaddle/Paddle/pull/37061), [#36945](https://github.com/PaddlePaddle/Paddle/pull/36945))

- 增强多线程场景下调试和报错功能，将子线程的报错捕获到主线程中统一抛出，以提升用户体验。([#36692](https://github.com/PaddlePaddle/Paddle/pull/36692)，[#36802](https://github.com/PaddlePaddle/Paddle/pull/36802))

#### 分布式训练

- 集合通信多机多卡训练基础功能
  
  - 新增弹性功能（含节点故障、扩容、缩容），提升分布式的容错能力。 ([#36684](https://github.com/PaddlePaddle/Paddle/pull/36684), [#37177](https://github.com/PaddlePaddle/Paddle/pull/37177), [#37781](https://github.com/PaddlePaddle/Paddle/pull/37781)) 
  
  - Launch启动模块，重构并新增 `master` 协同和节点个数 `nnodes` 定义 ，提升分布式启动易用性。 ([#40086](https://github.com/PaddlePaddle/Paddle/pull/40086), [#40568](https://github.com/PaddlePaddle/Paddle/pull/40568), [#40782](https://github.com/PaddlePaddle/Paddle/pull/40782), [#40844](https://github.com/PaddlePaddle/Paddle/pull/40844), [#40936](https://github.com/PaddlePaddle/Paddle/pull/40936), [#41190](https://github.com/PaddlePaddle/Paddle/pull/41190), [#41314](https://github.com/PaddlePaddle/Paddle/pull/41314)) 
  
  - 新增对 GPU/NPU/XPU 多种硬件的异构训练的支持。([#37613](https://github.com/PaddlePaddle/Paddle/pull/37613), [#37998](https://github.com/PaddlePaddle/Paddle/pull/37998)) 
  
  - 新增 fleet_executor 异步流水执行器。([#36966](https://github.com/PaddlePaddle/Paddle/pull/36966), [#37049](https://github.com/PaddlePaddle/Paddle/pull/37049), [#37087](https://github.com/PaddlePaddle/Paddle/pull/37087), [#37126](https://github.com/PaddlePaddle/Paddle/pull/37126), [#37150](https://github.com/PaddlePaddle/Paddle/pull/37150), [#37203](https://github.com/PaddlePaddle/Paddle/pull/37203), [#37167](https://github.com/PaddlePaddle/Paddle/pull/37167), [#37282](https://github.com/PaddlePaddle/Paddle/pull/37282), [#37319](https://github.com/PaddlePaddle/Paddle/pull/37319), [#37462](https://github.com/PaddlePaddle/Paddle/pull/37462), [#37507](https://github.com/PaddlePaddle/Paddle/pull/37507), [#37533](https://github.com/PaddlePaddle/Paddle/pull/37533), [#37576](https://github.com/PaddlePaddle/Paddle/pull/37576), [#37605](https://github.com/PaddlePaddle/Paddle/pull/37605), [#37691](https://github.com/PaddlePaddle/Paddle/pull/37691), [#37742](https://github.com/PaddlePaddle/Paddle/pull/37742), [#37783](https://github.com/PaddlePaddle/Paddle/pull/37783), [#37809](https://github.com/PaddlePaddle/Paddle/pull/37809), [#37862](https://github.com/PaddlePaddle/Paddle/pull/37862), [#37882](https://github.com/PaddlePaddle/Paddle/pull/37882), [#37934](https://github.com/PaddlePaddle/Paddle/pull/37934), [#38024](https://github.com/PaddlePaddle/Paddle/pull/38024), [#38083](https://github.com/PaddlePaddle/Paddle/pull/38083), [#38164](https://github.com/PaddlePaddle/Paddle/pull/38164), [#38261](https://github.com/PaddlePaddle/Paddle/pull/38261), [#38290](https://github.com/PaddlePaddle/Paddle/pull/38290), [#40607](https://github.com/PaddlePaddle/Paddle/pull/40607), [#37093](https://github.com/PaddlePaddle/Paddle/pull/37093), [#37106](https://github.com/PaddlePaddle/Paddle/pull/37106), [#37143](https://github.com/PaddlePaddle/Paddle/pull/37143), [#37338](https://github.com/PaddlePaddle/Paddle/pull/37338), [#37376](https://github.com/PaddlePaddle/Paddle/pull/37376), [#37485](https://github.com/PaddlePaddle/Paddle/pull/37485), [#37531](https://github.com/PaddlePaddle/Paddle/pull/37531), [#37623](https://github.com/PaddlePaddle/Paddle/pull/37623), [#37693](https://github.com/PaddlePaddle/Paddle/pull/37693), [#37755](https://github.com/PaddlePaddle/Paddle/pull/37755), [#37807](https://github.com/PaddlePaddle/Paddle/pull/37807), [#37889](https://github.com/PaddlePaddle/Paddle/pull/37889), [#38420](https://github.com/PaddlePaddle/Paddle/pull/38420), [#38539](https://github.com/PaddlePaddle/Paddle/pull/38539), [#36892](https://github.com/PaddlePaddle/Paddle/pull/36892), [#37084](https://github.com/PaddlePaddle/Paddle/pull/37084), [#37158](https://github.com/PaddlePaddle/Paddle/pull/37158), [#37361](https://github.com/PaddlePaddle/Paddle/pull/37361), [#37509](https://github.com/PaddlePaddle/Paddle/pull/37509), [#37603](https://github.com/PaddlePaddle/Paddle/pull/37603), [#37703](https://github.com/PaddlePaddle/Paddle/pull/37703), [#37824](https://github.com/PaddlePaddle/Paddle/pull/37824), [#38114](https://github.com/PaddlePaddle/Paddle/pull/38114), [#38322](https://github.com/PaddlePaddle/Paddle/pull/38322), [#38535](https://github.com/PaddlePaddle/Paddle/pull/38535), [#38650](https://github.com/PaddlePaddle/Paddle/pull/38650), [#38709](https://github.com/PaddlePaddle/Paddle/pull/38709), [#38799](https://github.com/PaddlePaddle/Paddle/pull/38799), [#38839](https://github.com/PaddlePaddle/Paddle/pull/38839), [#38904](https://github.com/PaddlePaddle/Paddle/pull/38904))
  
  - 新增分布式大模型推理功能。([#38795](https://github.com/PaddlePaddle/Paddle/pull/38795), [#39012](https://github.com/PaddlePaddle/Paddle/pull/39012), [#39032](https://github.com/PaddlePaddle/Paddle/pull/39032), [#39076](https://github.com/PaddlePaddle/Paddle/pull/39076), [#39194](https://github.com/PaddlePaddle/Paddle/pull/39194), [#39207](https://github.com/PaddlePaddle/Paddle/pull/39207), [#39241](https://github.com/PaddlePaddle/Paddle/pull/39241), [#39603](https://github.com/PaddlePaddle/Paddle/pull/39603), [#39758](https://github.com/PaddlePaddle/Paddle/pull/39758), [#39992](https://github.com/PaddlePaddle/Paddle/pull/39992)) 

- 动态图混合并行
  
  - 重构 `paddle.distributed.fleet.utils.recompute`，支持新动态图。 ([#41396](https://github.com/PaddlePaddle/Paddle/pull/41396))
  
  - 支持 Pure FP16 训练。([#36420](https://github.com/PaddlePaddle/Paddle/pull/36420)) 
  
  - 新增 MoE（Mixture of Experts）并行策略, 支持超大 MoE 模型训练。([#41092](https://github.com/PaddlePaddle/Paddle/pull/41092), [#40895](https://github.com/PaddlePaddle/Paddle/pull/40895), [#40850](https://github.com/PaddlePaddle/Paddle/pull/40580), [#39224](https://github.com/PaddlePaddle/Paddle/pull/39224))
  
  - 新增 GroupSharded 并行策略，支持 stage1、stage2、stage3三个阶段模型状态分组切片训练策略，支持同、异步通信，并可与 Recompute、AMP O1\O2、Offload、GroupShardedClipGrad、GroupShardedScaler 等基础功能组合使用。([#37489](https://github.com/PaddlePaddle/Paddle/pull/37489), [#37568](https://github.com/PaddlePaddle/Paddle/pull/37568), [#37707](https://github.com/PaddlePaddle/Paddle/pull/37707), [#37836](https://github.com/PaddlePaddle/Paddle/pull/37836), [#37947](https://github.com/PaddlePaddle/Paddle/pull/37947), [#38151](https://github.com/PaddlePaddle/Paddle/pull/38151), [#38407](https://github.com/PaddlePaddle/Paddle/pull/38407), [#38052](https://github.com/PaddlePaddle/Paddle/pull/38052), [#39112](https://github.com/PaddlePaddle/Paddle/pull/39112), [#38989](https://github.com/PaddlePaddle/Paddle/pull/38989), [#39171](https://github.com/PaddlePaddle/Paddle/pull/39171), [#39285](https://github.com/PaddlePaddle/Paddle/pull/39285), [#39334](https://github.com/PaddlePaddle/Paddle/pull/39334), [#39397](https://github.com/PaddlePaddle/Paddle/pull/39397), [#39581](https://github.com/PaddlePaddle/Paddle/pull/39581), [#39668](https://github.com/PaddlePaddle/Paddle/pull/39668), [#40129](https://github.com/PaddlePaddle/Paddle/pull/40129), [#40396](https://github.com/PaddlePaddle/Paddle/pull/40396), [#40488](https://github.com/PaddlePaddle/Paddle/pull/40488), [#40601](https://github.com/PaddlePaddle/Paddle/pull/40601)，[#37725](https://github.com/PaddlePaddle/Paddle/pull/37725)，[#37904](https://github.com/PaddlePaddle/Paddle/pull/37904), [#38064](https://github.com/PaddlePaddle/Paddle/pull/38064))

- 静态图混合并行
  
  - 新增`scale_gradient`标志位至`gradient_scale_configs`，用于控制流水线并行下梯度聚合运算对梯度进行求平均运算的位置。([#36384](https://github.com/PaddlePaddle/Paddle/pull/36384)) 
  
  - 张量模型并行下，dropout 支持设置确定性随机种子生成器，以确保非分布式变量的随机一致性和分布式变量的随机性。([#36228](https://github.com/PaddlePaddle/Paddle/pull/36228)) 
  
  - NPU 混合并行支持 Offload，可节约40%显存。([#37224](https://github.com/PaddlePaddle/Paddle/pull/37224))
  
  - 为 seed op 增加 `force_cpu` 可选参数，使 dropout 可以直接从 CPU 读取 seed 的值。([#35820](https://github.com/PaddlePaddle/Paddle/pull/35820)) 
  
  - 完善Automatic Sparsity (ASP)sharding策略，支持根据program选择sharding策略。(#[#40028](https://github.com/PaddlePaddle/Paddle/pull/40028)）

- 自动并行
  
  - 新增逻辑进程与物理设备自动映射后的进程重新启动（relaunch）。([#37523](https://github.com/PaddlePaddle/Paddle/pull/37523), [#37326](https://github.com/PaddlePaddle/Paddle/pull/37326))  
  
  - 完善自动并行底层机制和接口，利于各个模块统一和添加优化 pass。([#36617](https://github.com/PaddlePaddle/Paddle/pull/36617), [#38132](https://github.com/PaddlePaddle/Paddle/pull/38132)) 
  
  - 新增统一资源表示，支持逻辑进程与物理设备自动映射功能。([#37091](https://github.com/PaddlePaddle/Paddle/pull/37091), [#37482](https://github.com/PaddlePaddle/Paddle/pull/37482), [#37094](https://github.com/PaddlePaddle/Paddle/pull/37094)) 
  
  - 完善自动并行计算图反向和更新部分的分布式属性补全功能。([#36744](https://github.com/PaddlePaddle/Paddle/pull/36744))
  
  - 新增数据切分功能。([#36055](https://github.com/PaddlePaddle/Paddle/pull/36055)) 
  
  - 新增张量重切分功能，根据张量和算子的分布式属性对张量进行重新切分。([#40865](https://github.com/PaddlePaddle/Paddle/pull/40865), [#41106](https://github.com/PaddlePaddle/Paddle/pull/41106))
  
  - 新增资源数量或并行策略变化时分布式参数的自动转换功能。([#40434](https://github.com/PaddlePaddle/Paddle/pull/40434))
  
  - 新增梯度累加功能（GradientMerge），减少通信次数，提升训练效率。([#38259](https://github.com/PaddlePaddle/Paddle/pull/38259), [#40737](https://github.com/PaddlePaddle/Paddle/pull/40737)) 
  
  - 新增重计算功能(Recompute)，优化显存。([#38920](https://github.com/PaddlePaddle/Paddle/pull/38920)) 
  
  - 新增 Sharding 优化 pass， 支持 p-g-os 3 个stage 的切分优化。([#38502](https://github.com/PaddlePaddle/Paddle/pull/38502))
  
  - 新增 AMP + FP16 优化 pass。([#38764](https://github.com/PaddlePaddle/Paddle/pull/38764), [#40615](https://github.com/PaddlePaddle/Paddle/pull/40615))
  
  - 新增 Transformer 类模型的 QKV fuse 切分。([#39080](https://github.com/PaddlePaddle/Paddle/pull/39080))
  
  - 新增 while op 的分布式属性推导功能，确保迭代推导算法能收敛。([#39939](https://github.com/PaddlePaddle/Paddle/pull/39939), [#39086](https://github.com/PaddlePaddle/Paddle/pull/39086), [#39014](https://github.com/PaddlePaddle/Paddle/pull/39014)) 
  
  - 支持子 block 和 while op 控制流的训练和推理。([#39612](https://github.com/PaddlePaddle/Paddle/pull/39612), [#39895](https://github.com/PaddlePaddle/Paddle/pull/39895), [#40077](https://github.com/PaddlePaddle/Paddle/pull/40077))

- 参数服务器
  
  - GPUPS 下，新增 NAN/INF 值检查工具。 ([#38131](https://github.com/PaddlePaddle/Paddle/pull/38131))
  
  - GPUPS 下，新增 set_date 接口，适配增量训练。([#36194](https://github.com/PaddlePaddle/Paddle/pull/36194)) 
  
  - GPUPS 下，新增异步 release dataset 功能。 ([#37790](https://github.com/PaddlePaddle/Paddle/pull/37790))
  
  - GPUPS 下，支持 Dump 参数和中间层（[#36157](https://github.com/PaddlePaddle/Paddle/pull/36157)）；
  
  - GPUPS 下，支持优化器参数配置。([#39783](https://github.com/PaddlePaddle/Paddle/pull/39783), [#39849](https://github.com/PaddlePaddle/Paddle/pull/39849))
  
  - 统一参数服务器下，重构通信、存储等各个模块基类，提升各个模块的易二次开发性。([#41207](https://github.com/PaddlePaddle/Paddle/pull/41207), [#41022](https://github.com/PaddlePaddle/Paddle/pull/41022), [#40702](https://github.com/PaddlePaddle/Paddle/pull/40702), [#39341](https://github.com/PaddlePaddle/Paddle/pull/39341) [#39377](https://github.com/PaddlePaddle/Paddle/pull/39377), [#39191](https://github.com/PaddlePaddle/Paddle/pull/39191), [#39064](https://github.com/PaddlePaddle/Paddle/pull/39064))
  
  - 统一参数服务器下，新增评估指标模块，支持 AUC/WuAUC/MaskAuc 等评估指标计算及可自定义扩展。 ([#38789](https://github.com/PaddlePaddle/Paddle/pull/38789)) 

#### Profiler

- Python 层新增性能分析模块 `paddle.profiler`: 提供对训推过程中性能数据的收集，导出和统计的功能。 ([#40065](https://github.com/PaddlePaddle/Paddle/pull/40065), [#40357](https://github.com/PaddlePaddle/Paddle/pull/40357), [#40888](https://github.com/PaddlePaddle/Paddle/pull/40888)) 
  
  - `paddle.profiler.Profiler`，性能分析器，用户交互的接口。([#41029](https://github.com/PaddlePaddle/Paddle/pull/41029), [#41524](https://github.com/PaddlePaddle/Paddle/pull/41524), [#41157](https://github.com/PaddlePaddle/Paddle/pull/41157), [#40249](https://github.com/PaddlePaddle/Paddle/pull/40249), [#40111](https://github.com/PaddlePaddle/Paddle/pull/40111), [#39964](https://github.com/PaddlePaddle/Paddle/pull/39964), [#40133](https://github.com/PaddlePaddle/Paddle/pull/40133))
  
  - `paddle.profiler.RecordEvent`，提供自定义打点来记录时间的功能。 ([#39693](https://github.com/PaddlePaddle/Paddle/pull/39693), [#39694](https://github.com/PaddlePaddle/Paddle/pull/39694), [#39695](https://github.com/PaddlePaddle/Paddle/pull/39695), [#39675](https://github.com/PaddlePaddle/Paddle/pull/39675),[#41445](https://github.com/PaddlePaddle/Paddle/pull/41445), [#41132](https://github.com/PaddlePaddle/Paddle/pull/41132))
  
  - `paddle.profiler.ProfilerTarget`，指定性能分析的目标设备。
  
  - `paddle.profiler.ProfilerState`，表示性能分析器的状态。
  
  - `paddle.profiler.SortedKeys`，指定统计表单内数据的排序方式。
  
  - `paddle.profiler.make_scheduler`，生成性能分析器状态的调度器，实现采集范围的周期性控制。
  
  - `paddle.profiler.export_chrome_tracing`，将性能数据保存到可供 chrome://tracing 插件查看的 google chrome tracing 文件。 ([#39316](https://github.com/PaddlePaddle/Paddle/pull/39316), [#39984](https://github.com/PaddlePaddle/Paddle/pull/39984), [#41029](https://github.com/PaddlePaddle/Paddle/pull/41029))
  
  - `paddle.profiler.export_protobuf`，将性能数据保存到内部结构表示的 protobuf 文件。 ([#39519](https://github.com/PaddlePaddle/Paddle/pull/39519), [#39109](https://github.com/PaddlePaddle/Paddle/pull/39109), [#39474](https://github.com/PaddlePaddle/Paddle/pull/39474))
  
  - `paddle.profiler.load_profiler_result`，载入所保存到 protobuf 文件的性能数据。
  
  - `paddle.profiler.Profiler`通过指定 `timer_only` 参数，对模型进行数据读取、step 开销和吞吐量的统计。([#40386](https://github.com/PaddlePaddle/Paddle/pull/40386)) 

- C++层重构 Profiler 底层基础设施 
  
  - 重构 Profiler 的控制器架构。（[#38826](https://github.com/PaddlePaddle/Paddle/pull/38826), [#39230](https://github.com/PaddlePaddle/Paddle/pull/39230), [#39779](https://github.com/PaddlePaddle/Paddle/pull/39779) ）
  
  - 新增 Host Tracer，收集主机侧性能指标。（[#37629](https://github.com/PaddlePaddle/Paddle/pull/39629), [#37766](https://github.com/PaddlePaddle/Paddle/pull/37766), [#37944](https://github.com/PaddlePaddle/Paddle/pull/37944), [#38280](https://github.com/PaddlePaddle/Paddle/pull/38280), [#39975](https://github.com/PaddlePaddle/Paddle/pull/39975), [#40460](https://github.com/PaddlePaddle/Paddle/pull/40460)）
  
  - 新增 CUDA Tracer，收集设备侧性能指标。（[#39488](https://github.com/PaddlePaddle/Paddle/pull/39488)）
  
  - Profiler 支持分级。（[#39926](https://github.com/PaddlePaddle/Paddle/pull/39926)）

#### 其他

- 模型量化
  
  - 升级量化存储格式，并统一动、静态图量化格式。([#41041](https://github.com/PaddlePaddle/Paddle/pull/41041))
  
  - 新增离线量化方法: EMD、Adaround。([#40421](https://github.com/PaddlePaddle/Paddle/pull/40421), [#38460](https://github.com/PaddlePaddle/Paddle/pull/38460))
  
  - 支持更多 op 适配模 op 量化。([#40083](https://github.com/PaddlePaddle/Paddle/pull/40083))
  
  - 支持控制流中的OP量化。([#37498](https://github.com/PaddlePaddle/Paddle/pull/37498)) 
  
  - 新增支持matmul_v2 OP的量化。([#36469](https://github.com/PaddlePaddle/Paddle/pull/36469))
  
  - 新增支持量化后的 matmul_v2 在 TensorRT 上的推理。([#36594](https://github.com/PaddlePaddle/Paddle/pull/36594))

- 显存优化
  
  - 实现多 stream 安全 Allocator，支持在多 stream 异步计算场景下安全高效地使用显存。([#37290](https://github.com/PaddlePaddle/Paddle/pull/37290)) 
  
  - 新增运行时显存监控模块(paddle.device.cuda.max_memory_allocated, paddle.device.cuda.max_memory_reserved, paddle.device.cuda.memory_allocated and paddle.device.cuda.memory_reserved)，支持高性能地实时统计显存数据。([#38657](https://github.com/PaddlePaddle/Paddle/pull/38657)) 
  
  - 实现 CPU-GPU 统一内存寻址（CUDA Managed Memory），支持在显存受限场景下训练超大模型。([#39075](https://github.com/PaddlePaddle/Paddle/pull/39075)) 
  
  - C++底层新增GetBasePtr接口，用来获取设备接口CUDAMalloc创建的设备地址。([#37978](https://github.com/PaddlePaddle/Paddle/pull/37978)) 
  
  - 减少AutoGrowth Allocator 中 free blocks 的数量，提升显存分配性能。([#35732](https://github.com/PaddlePaddle/Paddle/pull/35732)) 
  
  - 对于 `initializer.Normal` 和 `initializer.Constant` 数据类型是 FP16 的 Tensor 去除多余的 float32 临时 Tensor 以及 cast，节省2倍显存。 ([#38818](https://github.com/PaddlePaddle/Paddle/pull/38818)) 

- 动态图高阶导数组网测试 
  
  - 为动态图增加三阶导数组网测试，以及Broadcast情况的测试。 ([#36814](https://github.com/PaddlePaddle/Paddle/pull/36814) , [#37377](https://github.com/PaddlePaddle/Paddle/pull/37377))

- 自定义 op：支持 ROCm(HIP) 平台进行自定义 op 注册。 ([#36771](https://github.com/PaddlePaddle/Paddle/pull/36771)) 

- Cost Model：增加基于运行 Profile 的 Cost Model。 ([#35774](https://github.com/PaddlePaddle/Paddle/pull/35774))

- 提供定制化层 (nn.Layer)的自动稀疏训练支持，让用戶可根据自定义的Prune函数来对其设计的层进行稀疏剪枝。([#40253](https://github.com/PaddlePaddle/Paddle/pull/40253)) 

- 新增字符串张量底层数据结构表示，使框架具备字符串张量表示和计算的能力。([#39830](https://github.com/PaddlePaddle/Paddle/pull/39830), [#40992](https://github.com/PaddlePaddle/Paddle/pull/40992)) 

- 新增或者升级 oneDNN FP32/int8/bfloat16 Kernel，包括：
  
  - ELU ([#37149](https://github.com/PaddlePaddle/Paddle/pull/37149))
  
  - exp ([#38624](https://github.com/PaddlePaddle/Paddle/pull/38624))
  
  - stack ([#37002](https://github.com/PaddlePaddle/Paddle/pull/37002))
  
  - softplus ([#36382](https://github.com/PaddlePaddle/Paddle/pull/36382))
  
  - round ([#39653](https://github.com/PaddlePaddle/Paddle/pull/39653))
  
  - shape ([#36033](https://github.com/PaddlePaddle/Paddle/pull/36033))
  
  - flatten and flatten2 ([#35892](https://github.com/PaddlePaddle/Paddle/pull/35892))
  
  - slice ([#37630](https://github.com/PaddlePaddle/Paddle/pull/37630))
  
  - elementwise_mul ([#40546](https://github.com/PaddlePaddle/Paddle/pull/40546))
  
  - elementwise_add ([#38176](https://github.com/PaddlePaddle/Paddle/pull/38176))
  
  - ementwise_div ([#36158](https://github.com/PaddlePaddle/Paddle/pull/36158))
  
  - elementwise_sub ([#35662](https://github.com/PaddlePaddle/Paddle/pull/35662))
  
  - roi_align ([#37848](https://github.com/PaddlePaddle/Paddle/pull/37848))
  
  - nearest_interp and nearest_interp_v2 ([#37985](https://github.com/PaddlePaddle/Paddle/pull/37985)，[#38622](https://github.com/PaddlePaddle/Paddle/pull/38622)，[#39490](https://github.com/PaddlePaddle/Paddle/pull/39490))
  
  - assembly optimized Adam ([#39158](https://github.com/PaddlePaddle/Paddle/pull/39158))
  
  - logsoftmax ([#39793](https://github.com/PaddlePaddle/Paddle/pull/39793))
  
  - activation ([#40721](https://github.com/PaddlePaddle/Paddle/pull/40721))
  
  - mul ([#38552](https://github.com/PaddlePaddle/Paddle/pull/38552))
  
  - mean ([#37104](https://github.com/PaddlePaddle/Paddle/pull/37104))
  
  - relu ([#36265](https://github.com/PaddlePaddle/Paddle/pull/36265))
  
  - pool2d ([#37081](https://github.com/PaddlePaddle/Paddle/pull/37081))
  
  - concat ([#35889](https://github.com/PaddlePaddle/Paddle/pull/35889))
  
  - conv2d ([#38507](https://github.com/PaddlePaddle/Paddle/pull/38507)，[#38938](https://github.com/PaddlePaddle/Paddle/pull/38938)，[#36284](https://github.com/PaddlePaddle/Paddle/pull/36284))
  
  - LayerNorm ([#40418](https://github.com/PaddlePaddle/Paddle/pull/40418))

### （2）功能优化

#### API

- 为 `paddle.Model`新增支持混合精度训练 O2 模式，即支持原来动/静态图的 Pure FP16 训练模式。([#36441](https://github.com/PaddlePaddle/Paddle/pull/40962441)) 

- 为 `paddle.nn.Layer` 支持 self chain 调用。([#36609](https://github.com/PaddlePaddle/Paddle/pull/36609)) 

- 为 `paddle.nn.Layer`的`to`方法添加`is_distributed`属性的设置，保证网络参数转换前后分布式属性保持一致。([#36221](https://github.com/PaddlePaddle/Paddle/pull/36221)) 

- 完善 `paddle.nn.Layer`的`to` 方法的参数转换逻辑，降低转换过程占用的峰值显存，提高转换成功率。([#36862](https://github.com/PaddlePaddle/Paddle/pull/36862)) 

- 为 `paddle.incubate.graph_send_recv`支持设置输出 Tensor 的 shape，有利于减少实际计算过程的显存占用。([#40509](https://github.com/PaddlePaddle/Paddle/pull/40509)) 

- 为 `paddle.incubate.segment_sum`、`segment_mean`、`segment_max`、`segment_min` 新增 int32、int64 数据类型支持。([#40577](https://github.com/PaddlePaddle/Paddle/pull/40577)) 

- 为 transpose op 新增 bool 类型支持。([#35886](https://github.com/PaddlePaddle/Paddle/pull/35886)) 

- 将 `paddle.mm` 底层算子从 matmul 切换到matmul_v2。 ([#35770](https://github.com/PaddlePaddle/Paddle/pull/35770)) 

- 为 `paddle.einsum` 支持静态图模式调用，支持未知 shape。 ([#40360](https://github.com/PaddlePaddle/Paddle/pull/40360)) 

- 为 `paddle.nn.functional.margin_cross_entropy` 和 `paddle.nn.functional.class_center_sample` 支持数据并行。([#39852](https://github.com/PaddlePaddle/Paddle/pull/39852)) 

- 为 `paddle.nn.functional.grid_sample`支持形状为[1]的输入。（[#36183](https://github.com/PaddlePaddle/Paddle/pull/36183)）

- 为 `paddle.nn.PRelu` 支持 `NHWC` 数据格式。([#37019](https://github.com/PaddlePaddle/Paddle/pull/37019)) 

- 为 `paddle.nn.functional.class_center_sample` 支持使用 `paddle.seed` 固定随机状态。([#38248](https://github.com/PaddlePaddle/Paddle/pull/38248)) 

- 为 `paddle.fft` 下所有 API 新增 ROCM 后端支持，并优化 CUFFT 后端报错信息。([#36415](https://github.com/PaddlePaddle/Paddle/pull/36415), [#36114](https://github.com/PaddlePaddle/Paddle/pull/36114/files))

- 为 `Tensor.getitem` 增加对切片部分维度为0的功能支持，即允许切片索引结果为空。([#37313](https://github.com/PaddlePaddle/Paddle/pull/37313)) 

- 为 `Tensor.setitem` 支持 int 和 bool 类型 Tensor 使用 bool 索引。([#37761](https://github.com/PaddlePaddle/Paddle/pull/37761)) 

- 为 `paddle.nn.functional.interpolate` 支持 nearest 模式时输入 shape 为 5D。([#38868](https://github.com/PaddlePaddle/Paddle/pull/38868)) 

- 为 `paddle.nn.Embedding`、`paddle.gather` 增加 int16 支持。([#40964](https://github.com/PaddlePaddle/Paddle/pull/40964), [#40052](https://github.com/PaddlePaddle/Paddle/pull/40052)) 

- 为 `paddle.distributed.spawn`添加 CPU 单机数据并行。 ([#35745](https://github.com/PaddlePaddle/Paddle/pull/35745), [#36758](https://github.com/PaddlePaddle/Paddle/pull/36758), [#36637](https://github.com/PaddlePaddle/Paddle/pull/36637)) 

- 新增`depthwise_conv2d`MKLDNN 算子。([#38484](https://github.com/PaddlePaddle/Paddle/pull/38484)) 

- 为`paddle.abs`、`paddle.transpose`、`paddle.squeeze`、`paddle.unsqueeze`、 `paddle.matmul`、`paddle.full` 静态图数据类型检测中增加复数类型。([#40113](https://github.com/PaddlePaddle/Paddle/pull/40113))

- 为 `paddle.autograd.PyLayer` 支持 tuple/list 类型的参数。([#38146](https://github.com/PaddlePaddle/Paddle/pull/38146))

- 为 `paddle.autograd.PyLayer` 增加检查 inplace 策略下，输入叶子节点的 Tensor 的检查报错机制。([#37931](https://github.com/PaddlePaddle/Paddle/pull/37931))

- 为 `paddle.autograd.PyLayer` 支持 HIP 库。([#38184](https://github.com/PaddlePaddle/Paddle/pull/38184)) 

- 为 `paddle.take_along_axis`、`paddle.put_along_axis` 支持更多 size 的输入，允许 index 矩阵的 shape size 大于 arr 矩阵的 shape size。 ([#39072](https://github.com/PaddlePaddle/Paddle/pull/39072))

- 优化 API `paddle.nn.Pad2D`在 replicate 为0时的报错信息。([#36510](https://github.com/PaddlePaddle/Paddle/pull/36510/files))

- 支持 API `paddle.nn.Pad2D`在 tuple 格式的 pad 输入。([#35985](https://github.com/PaddlePaddle/Paddle/pull/35985/files))

- 新增 `paddle.distributed.InMemoryDataset` 中 tdm_sample API 以支持 TDM 算法中的采样操作。([#37044](https://github.com/PaddlePaddle/Paddle/pull/37044)) 

- 新增对于`paddle.jit.save`的 Pre-saving Hooks 机制。([#38186](https://github.com/PaddlePaddle/Paddle/pull/38186)）

- 新增高阶微分相关 API：
  
  - `elementwise_add` 增加三阶 Kernel，支持三阶微分的计算。([#36508](https://github.com/PaddlePaddle/Paddle/pull/36508), [#36618](https://github.com/PaddlePaddle/Paddle/pull/36618)) 
  
  - `matmul_v2` 增加三阶 Kernel，支持三阶微分的计算。([#36459](https://github.com/PaddlePaddle/Paddle/pull/36459))
  
  - `elementwise_mul` 增加三阶 Kernel，支持三阶微分的计算。 ([#37152](https://github.com/PaddlePaddle/Paddle/pull/37547)) 

- 完善`paddle.amp.GradScaler`调用 check_finite_and_unscale op 的逻辑，消除该处创建 bool 变量所引入的 cudaMemcpy。([#37770](https://github.com/PaddlePaddle/Paddle/pull/37770))

- 新增对 unstack 和 unique op 元素个数为0的 Tensor 增加检查。([#36021](https://github.com/PaddlePaddle/Paddle/pull/36021)) 

#### IR(Intermediate Representation)

- 动态图转静态图
  
  - 优化动转静下 `ProgramCache.last` 接口行为，使其返回最近使用的 Program，而非最后生成的Program。([#39541](https://github.com/PaddlePaddle/Paddle/pull/39541)) 
  
  - 优化动转静下 `paddle.reshape` API 的报错信息，新增推荐用法提示。([#40599](https://github.com/PaddlePaddle/Paddle/pull/40599)) 
  
  - 优化动转静代码转写时 `is_api_in_module` 函数中异常捕获类型。([#40243](https://github.com/PaddlePaddle/Paddle/pull/40243)) 
  
  - 优化动转静模块报错提示，默认隐藏warning信息。([#39730](https://github.com/PaddlePaddle/Paddle/pull/https://github.com/PaddlePaddle/Paddle/pull/39730)) 
  
  - 增加动转静对于type hint语法的支持，提高变量类型分析的准确性。([#39572](https://github.com/PaddlePaddle/Paddle/pull/39572)) 
  
  - 优化 `paddle.cond` 功能，允许bool、int等基本类型支持值相等。([#37888](https://github.com/PaddlePaddle/Paddle/pull/37888)) 
  
  - 优化动转静`@to_static` 装饰普通函数时，允许切换train/eval模式。([#37383](https://github.com/PaddlePaddle/Paddle/pull/37383)) 
  
  - 优化动转静报错栈，突出用户相关代码，减少框架冗余报错栈。([#36741](https://github.com/PaddlePaddle/Paddle/pull/36741)) 
  
  - 移除`paddle.cond` 返回值中 `no_value` 占位符。([#36513](https://github.com/PaddlePaddle/Paddle/pull/36513)、[#36826](https://github.com/PaddlePaddle/Paddle/pull/36826)) 
  
  - 为动转静 run_program op 适配新动态图模式。([#40198](https://github.com/PaddlePaddle/Paddle/pull/40198), [#40355](https://github.com/PaddlePaddle/Paddle/pull/40355)) 
  
  - 新增对于 zip 语法的检查。 ([#37846](https://github.com/PaddlePaddle/Paddle/pull/https://github.com/PaddlePaddle/Paddle/pull/37846)) 
  
  - 修复 `paddle.signal.frame`、`paddle.signal.stft`、`paddle.signal.istft` 因维度和类型判断错误导致的动转静失败问题。([#40113](https://github.com/PaddlePaddle/Paddle/pull/40113))
  
  - 为 mean、pad3d ops 新增注册复数类型 Kernel。([#40113](https://github.com/PaddlePaddle/Paddle/pull/40113))

#### 混合精度训练

- 为 amp 添加 GPU Compute Capability 环境检查，对无法产生训练加速效果的 GPU 环境添加使用警告。([#38086](https://github.com/PaddlePaddle/Paddle/pull/38086)) 

- 添加`paddle.amp.decorate`与`paddle.DataParallel`同时使用时调用顺序的检查。([#38785](https://github.com/PaddlePaddle/Paddle/pull/38785)) 

#### 分布式训练

- 分布式训练基础功能
  
  - 优化 Fleet API 和 DistributedStrategy 配置以使用动态图并行功能，提升动态图易用性。([#40408](https://github.com/PaddlePaddle/Paddle/pull/40408))
  
  - 优化动态图混合并行 HybridParallelClipGrad 策略，支持4D混合并行 + Pure FP16 训练。([#36237](https://github.com/PaddlePaddle/Paddle/pull/36237), [#36555](https://github.com/PaddlePaddle/Paddle/pull/36555)) 
  
  - 重构动态图数据并行策略，以支持新动态图和新通信库功能。([#40389](https://github.com/PaddlePaddle/Paddle/pull/40389), [#40593](https://github.com/PaddlePaddle/Paddle/pull/40593), [#40836](https://github.com/PaddlePaddle/Paddle/pull/40836), [#41119](https://github.com/PaddlePaddle/Paddle/pull/41119), [#41413](https://github.com/PaddlePaddle/Paddle/pull/41413), [#39987](https://github.com/PaddlePaddle/Paddle/pull/39987))  
  
  - 为 fused_attention op 支持分布式张量模型并行。([#40101](https://github.com/PaddlePaddle/Paddle/pull/40101)) 
  
  - 为 fused_feedforward op 支持分布式张量模型并行。([#40160](https://github.com/PaddlePaddle/Paddle/pull/40160)) 

- 图检索引擎
  
  - 优化图引擎的图采样接口返回的数据格式，采样速度提升3倍。([#37315](https://github.com/PaddlePaddle/Paddle/pull/37315)) 
  
  - 减少图引擎线程量以提升性能。([#37098](https://github.com/PaddlePaddle/Paddle/pull/37098)) 
  
  - 优化图引擎数据传输以提升性能。([#37341](https://github.com/PaddlePaddle/Paddle/pull/37341))
  
  - 利用模型中 embedding op 的拓扑关系，优化 embedding op 的合并逻辑以提升性能。[(#35942)](https://github.com/PaddlePaddle/Paddle/pull/35942) 

- 通信库：重构通信库，提升通信库的易扩展性和二次开发性，支持异构通信。 ([#41398](https://github.com/PaddlePaddle/Paddle/pull/41398), [#39720](https://github.com/PaddlePaddle/Paddle/pull/39720), [#40911](https://github.com/PaddlePaddle/Paddle/pull/40911), [#40579](https://github.com/PaddlePaddle/Paddle/pull/40579), [#40629](https://github.com/PaddlePaddle/Paddle/pull/40629), [#40437](https://github.com/PaddlePaddle/Paddle/pull/40437), [#40430](https://github.com/PaddlePaddle/Paddle/pull/40430), [#40228](https://github.com/PaddlePaddle/Paddle/pull/40228), [#40181](https://github.com/PaddlePaddle/Paddle/pull/40181), [#40100](https://github.com/PaddlePaddle/Paddle/pull/40100), [#40097](https://github.com/PaddlePaddle/Paddle/pull/40097), [#39892](https://github.com/PaddlePaddle/Paddle/pull/39892), [#39384](https://github.com/PaddlePaddle/Paddle/pull/39384), [#39737](https://github.com/PaddlePaddle/Paddle/pull/39737), [#40040](https://github.com/PaddlePaddle/Paddle/pull/40040)) 

#### 其他

- 报错调试优化
  
  - 优化 cross_entropy op 对 `label` 的边界检查报错信息。([#40001](https://github.com/PaddlePaddle/Paddle/pull/40001)) 
  
  - 为动态图添加 op 执行时`infer_shape`和`compute`方法的 profile record，用于在 timeline 中展示其开销。([#39023](https://github.com/PaddlePaddle/Paddle/pull/39023)) 
  
  - 替换了 Windows 下容易出现未知异常的 `pybind::index_error` 报错提示。([#40538](https://github.com/PaddlePaddle/Paddle/pull/40538)) 
  
  - 添加用户 scatter op 越界检查的报错信息。([#37429](https://github.com/PaddlePaddle/Paddle/pull/37429))

- 下载工具：针对`paddle.utils.download.get_path_from_url`中解压含多文件目录速度慢的问题，将原先循环遍历目录下文件逐一解压的方式替换为在目录上调用 extractall 一次解压的方式，解压速度大幅提升。([#37311](https://github.com/PaddlePaddle/Paddle/pull/37311))

- 加速 `fake_quantize_range_abs_max`、`fake_quantize_abs_max`、`fake_quantize_dequantize_abs_max`、 `fake_quantize_moving_average_abs_max` 等量化训练。([#40491](https://github.com/PaddlePaddle/Paddle/pull/40491))

### （3）性能优化

#### 分布式训练

- 混合并行优化器 sharding 支持 optimize_cast 优化，将前反向参数 cast 移到优化器阶段，性能提升7%。([#35878](https://github.com/PaddlePaddle/Paddle/pull/35878)) 

- GPUPS 优化：支持梯度 fuse allreduce 训练，训练提升20%。 ([#35131](https://github.com/PaddlePaddle/Paddle/pull/35131))

- GPUPS 优化：dump CPU 优化提速3.21倍。 ([#40068](https://github.com/PaddlePaddle/Paddle/pull/40068)) 

- CPU 参数服务器流式训练优化：支持稀疏参数统计量自动统计、稀疏参数增量保存等功能，训练性能提升20%。([#36465](https://github.com/PaddlePaddle/Paddle/pull/36465), [#36601](https://github.com/PaddlePaddle/Paddle/pull/36601), [#36734](https://github.com/PaddlePaddle/Paddle/pull/36734), [#36909](https://github.com/PaddlePaddle/Paddle/pull/36909), [#36943](https://github.com/PaddlePaddle/Paddle/pull/36943), [#37181](https://github.com/PaddlePaddle/Paddle/pull/37181), [#37194](https://github.com/PaddlePaddle/Paddle/pull/37194), [#37515](https://github.com/PaddlePaddle/Paddle/pull/37515), [#37626](https://github.com/PaddlePaddle/Paddle/pull/37626), [#37995](https://github.com/PaddlePaddle/Paddle/pull/37995), [#38582](https://github.com/PaddlePaddle/Paddle/pull/38582), [#39250](https://github.com/PaddlePaddle/Paddle/pull/39250), [#40762](https://github.com/PaddlePaddle/Paddle/pull/40762), [#41234](https://github.com/PaddlePaddle/Paddle/pull/41234), [#41320](https://github.com/PaddlePaddle/Paddle/pull/41320), [#41400](https://github.com/PaddlePaddle/Paddle/pull/41400)) 

#### 算子优化

- 优化 `FasterTokenizer` 性能，性能与优化前相比提升10%。 ([#36701](https://github.com/PaddlePaddle/Paddle/pull/36701)) 

- 优化 `index_select` 反向计算，性能较优化前有3.7~25.2倍提升。([#37055](https://github.com/PaddlePaddle/Paddle/pull/37055)) 

- 优化 `paddle.nn.ClipByGlobalNorm` 的性能，以10*10的 `paddle.nn.Linear` 为例，性能与优化前相比提升30%左右。 ([#38209](https://github.com/PaddlePaddle/Paddle/pull/38209)) 

- 优化 `pnorm` 在 `axis` 维度极大或极小情况下的性能，前向速度提升31~96倍，反向速度提升1.1~19倍。([#37685](https://github.com/PaddlePaddle/Paddle/pull/37685), [#38215](https://github.com/PaddlePaddle/Paddle/pull/38215), [#39011](https://github.com/PaddlePaddle/Paddle/pull/39011)) 

- 优化 `softmax` 前、反向性能，对于 `axis!=-1` 的配置加速比为2倍左右。([#38602](https://github.com/PaddlePaddle/Paddle/pull/38602), [#38609](https://github.com/PaddlePaddle/Paddle/pull/38609), [#32387](https://github.com/PaddlePaddle/Paddle/pull/32387), [#37927](https://github.com/PaddlePaddle/Paddle/pull/37927/files))

- 优化 `log_softmax` 前、反向性能，对于 `axis!=-1`的配置加速比为6~20倍左右。([#38992](https://github.com/PaddlePaddle/Paddle/pull/38992), [#40612](https://github.com/PaddlePaddle/Paddle/pull/40612)) 

- 优化 `softmax_with_cross_entropy` 前、反向性能，对于 `hard_label` 的配置加速比为1.3倍左右。([#39553](https://github.com/PaddlePaddle/Paddle/pull/39553), [#40424](https://github.com/PaddlePaddle/Paddle/pull/40424), [#40643](https://github.com/PaddlePaddle/Paddle/pull/40643)) 

- 优化 `top_k` 性能，对于一维且 `k` 较大时(k=5000)的配置加速比为22倍以上。([#40941](https://github.com/PaddlePaddle/Paddle/pull/40941)) 

- 优化 `elementwise_mul` 反向计算，较优化前有1.85~12.16倍性能提升。([#37728](https://github.com/PaddlePaddle/Paddle/pull/37728)) 

- 优化 `elementwise_min` 反向和 `elementwise_max` 反向，较优化前打平或有1.05~18.75倍性能提升。([#38236](https://github.com/PaddlePaddle/Paddle/pull/38236), [#37906](https://github.com/PaddlePaddle/Paddle/pull/37906)) 

- 优化 `nearest_interp` 前向和反向计算，前向较优化前性能有1.5~2.3倍提升；反向性能较优化前有60%~1.8倍提升。([#38528](https://github.com/PaddlePaddle/Paddle/pull/38528), [#39067](https://github.com/PaddlePaddle/Paddle/pull/39067)) 

- 优化 `bilinear_interp` 前向和反向计算，前向较优化前性能有0.4~2.3倍提升；反向性能较优化前有10%~30%提升。([#39243](https://github.com/PaddlePaddle/Paddle/pull/39243), [#39423](https://github.com/PaddlePaddle/Paddle/pull/39423)) 

- 优化 `dropout` 前向和反向计算，性能提升约20%。([#39795](https://github.com/PaddlePaddle/Paddle/pull/39795), [#38859](https://github.com/PaddlePaddle/Paddle/pull/38859), [#38279](https://github.com/PaddlePaddle/Paddle/pull/38279), [#40053](https://github.com/PaddlePaddle/Paddle/pull/40053))  

- 优化 `grid_sampler`前向和反向计算，前向较优化前性能有10%~30%提升；反向性能较优化前有10%~60%提升。([#39751](https://github.com/PaddlePaddle/Paddle/pull/39751)) 

- 优化 `group_norm` 前向和反向计算，前向性能提升1.04~2.35倍，反向性能提升1.12~1.18倍。([#39944](https://github.com/PaddlePaddle/Paddle/pull/39944), [#40657](https://github.com/PaddlePaddle/Paddle/pull/40657), [#39596](https://github.com/PaddlePaddle/Paddle/pull/39596))

- 优化 `conv1d` 前向和反向计算，前向性能提升1.00~2.01倍，反向性能提升1.01~474.56倍。([#38425](https://github.com/PaddlePaddle/Paddle/pull/38425)) 

- 优化 `elementwise_div` 反向计算，反向性能提升1.02~29.25倍。([#38044](https://github.com/PaddlePaddle/Paddle/pull/38044)) 

- 优化 `gelu` 前向和反向计算，前向性能提升1.13~1.43倍，反向性能提升1.10～1.55倍。([#38188](https://github.com/PaddlePaddle/Paddle/pull/38188), [#38263](https://github.com/PaddlePaddle/Paddle/pull/38263)) 

- 优化 `elementwise_sub` 反向计算，反向性能提升1.04~15.64倍。([#37754](https://github.com/PaddlePaddle/Paddle/pull/37754)) 

- 优化 `flip` 在输入一维数据时前向性能，性能提升100%。([#37825](https://github.com/PaddlePaddle/Paddle/pull/37825))

- 优化 `layer_norm` 前向和反向计算，前向较优化前提升2-5倍，反向较优化前提升20%~50%。([#39167](https://github.com/PaddlePaddle/Paddle/pull/39167), [#39247](https://github.com/PaddlePaddle/Paddle/pull/39247))

- 优化 `embedding` 前向和反向计算，前向较优化前最大提升1.51倍，反向较优化前提升1.03~7.79倍。([#39856](https://github.com/PaddlePaddle/Paddle/pull/39856), [#39886](https://github.com/PaddlePaddle/Paddle/pull/398866))

- 优化 `gelu` FP16 前向和反向计算，前向较优化前提升9%~12%，反向较优化前提升2%~9%。([#38980](https://github.com/PaddlePaddle/Paddle/pull/38980))

- 移除 `gather_nd`前反向算子中的 CPU -> GPU 显式数据传输操作，移除 `index_select` 前反向算子中的显式同步操作，将 `scatter_nd` 中的 GPU -> GPU 数据传输由同步操作改成异步操作。([#40933](https://github.com/PaddlePaddle/Paddle/pull/40933)) 

- 优化 `Lars optimzier` 计算，优化后 Resnet50 PF16 模型训练性能较优化前提升5.1%。 ([#35652](https://github.com/PaddlePaddle/Paddle/pull/35652), [#35476](https://github.com/PaddlePaddle/Paddle/pull/35476)) 

- 优化 `AvgPool2dGrad` 计算，优化后性能较优化前提升2.6倍。 ([#35389](https://github.com/PaddlePaddle/Paddle/pull/35389)) 

- 优化 `Elementwise` 类计算对于多元输出的功能支持，优化后计算性能较优化前提升最多可达15% 。（[#38329](https://github.com/PaddlePaddle/Paddle/pull/38329), [#38410](https://github.com/PaddlePaddle/Paddle/pull/38410)） 

- 优化 `Categorical`的 `probs`计算，简化计算逻辑，性能提升 4 ~ 5 倍。([#42178](https://github.com/PaddlePaddle/Paddle/pull/42178)) 

### （4）问题修复

#### API

- 修复 `paddle.sum` 输入参数类型和输出参数类型不一致且 `axis` 轴对应的 reduce 元素个数为1时，输出类型错误问题。([#36123](https://github.com/PaddlePaddle/Paddle/pull/36123)) 

- 修复 `paddle.flops` 在 layer 输出类型为 tuple 时的 `AttributeError`。([#38850](https://github.com/PaddlePaddle/Paddle/pull/38850))

- 修复 `paddle.diag` 因为没有反向 Kernel 而无法传播梯度的问题。([#40447](https://github.com/PaddlePaddle/Paddle/pull/40447)) 

- 修复 `paddle.sort` 输入存在 NaN 值排序错误。 ([#41070](https://github.com/PaddlePaddle/Paddle/pull/41070)) 

- 修复 `paddle.full_like` 输入存在 Inf 值构建 Tensor 错误。 ([#40232](https://github.com/PaddlePaddle/Paddle/pull/40232)) 

- 修复 `paddle.strided_slice` 在输入 starts 中数据小于 -rank 时，strided_slice 结果与 slice 不一致的 bug。 ([#39066](https://github.com/PaddlePaddle/Paddle/pull/39066)) 

- 修复 `max_pool` 系列算子在返回 index 时 infer_shape 计算错误的问题，受影响的 API 有 `paddle.nn.functional.max_pool1d/2d/3d`, `paddle.nn.functional.adaptive_max_pool1d/2d/3d`, `paddle.nn.MaxPool1D/2D/3D`, `paddle.nn.AdaptiveMaxPool1D/2D/3D`。([#40139](https://github.com/PaddlePaddle/Paddle/pull/40139))

- 修复 `max_pool` 系列算子返回的 pooling_mask 的 dtype 错误的问题，现在 pooling_mask 的 dtype 为 int32，受影响的 API 有 `paddle.nn.functional.max_pool1d/2d/3d`, `paddle.nn.functional.adaptive_max_pool1d/2d/3d`, `paddle.nn.MaxPool1D/2D/3D`, `paddle.nn.AdaptiveMaxPool1D/2D/3D`。([#39314](https://github.com/PaddlePaddle/Paddle/pull/39314))

- 修复 `paddle.shape` 默认存在反向梯度导致计算错误的问题。([#37340](https://github.com/PaddlePaddle/Paddle/pull/37340)) 

- 修复 `paddle.nn.Layer` 的 `to` 方法同时转换 dtype 和 place 存在的 bug。([#37007](https://github.com/PaddlePaddle/Paddle/pull/38007)) 

- 修复 `paddle.amp.decorate` 无法对非叶子网络层的参数改写为 FP16 的 bug。([#38402](https://github.com/PaddlePaddle/Paddle/pull/38402)) 

- 修复 `paddle.amp.decorate` 将 `paddle.nn.BatchNorm1D`、`paddle.nn.BatchNorm2D`、`paddle.nn.BatchNorm3D` 非输入参数改写为 FP16 的 bug。([#38541](https://github.com/PaddlePaddle/Paddle/pull/38541)) 

- 修复 `paddle.amp.decorate` 将 `paddle.nn.SyncBatchNorm` 非输入参数改写为 FP16 的 bug。([#40943](https://github.com/PaddlePaddle/Paddle/pull/40943)) 

- 修复 `paddle.nn.Layer.to` 当中多余的 warning。([#36700](https://github.com/PaddlePaddle/Paddle/pull/36700))

- 修复 `paddle.nn.RNN` 在控制流下使用报错的问题。([#41162](https://github.com/PaddlePaddle/Paddle/pull/41162)) 

- 修复 `paddle.to_tensor` 无法指定 Tensor 的 CUDA Place 的问题。([#39662](https://github.com/PaddlePaddle/Paddle/pull/39662)) 

- 修复 `paddle.nn.Identity` 没有公开的问题。([#39615](https://github.com/PaddlePaddle/Paddle/pull/39615))

- 修复动态图重构后，`fill_` 和 `zero_` inplace API的输入在 CUDAPinned Place上时，输出值不正确的 bug。([#41229](https://github.com/PaddlePaddle/Paddle/pull/41229)) 

- 动态图重构后，修复使用 append op 的方式调用 assign op 导致输出 Tensor 的 inplace version 值不正确的bug，修改为使用 `_C_ops` 的方式调用 assign op。([#41118](https://github.com/PaddlePaddle/Paddle/pull/41118)) 

- 移除 `elementwise_add` 三阶 Kernel 中不合理的代码，修复组网过程未初始化问题。 ([#36618](https://github.com/PaddlePaddle/Paddle/pull/36618)) 

- 修复 `conv2d` 执行 cuDNN Kernel 时属性缺失的问题。([#38827](https://github.com/PaddlePaddle/Paddle/pull/38827)) 

- 修复 `multiclass_nms3` 输出 shape 不正确的问题。([#40059](https://github.com/PaddlePaddle/Paddle/pull/40059)) 

- 修复 `yolo_box` 输出 shape 不正确的问题。([#40056](https://github.com/PaddlePaddle/Paddle/pull/40056)) 

- 修复高阶微分 `gradients` 接口在指定 target_grad 时未按预期生效的问题。([#40940](https://github.com/PaddlePaddle/Paddle/pull/40940/)) 

- 修复动态图 op`_BatchNormBase` 基类中修改了 default_dtype，导致后续组网参数类型错误的问题，受影响的API有 `paddle.nn.BatchNorm1D`，`paddle.nn.BatchNorm2D`，`paddle.nn.BatchNorm3D`，`paddle.nn.SyncBatchNorm`。具体原因是当 `get_default_dtype() == 'float16'` 时，通过 `set_default_dtype('float32')`修改默认参数数据类型，动态图组网的参数类型是通过 default_dtype 来创建的，因此当默认参数类型被修改后导致后续的组网参数类型错误。 ([#36376](https://github.com/PaddlePaddle/Paddle/pull/36376)) 

- 修复 batchnorm op 中，当数据类型为 FP32 ，且数据维度 `dims = 2，data_layout = NHWC` 时，反向 op 内中间变量未定义问题。 ([#37020](https://github.com/PaddlePaddle/Paddle/pull/37020)) 

- 修复静态图模式下，`paddle.static.nn.prelu` 对于 `NHWC` 输入格式且 `mode==channel` 权重的 shape 错误问题。([#38310](https://github.com/PaddlePaddle/Paddle/pull/38310)) 

- 修复多机情况下，`paddle.nn.functional.class_center_sample` CUDA 种子设置 bug。([#38815](https://github.com/PaddlePaddle/Paddle/pull/38815)) 

- 修复 `paddle.nn.functional.one_hot` 在输入不正确参数时，CUDA 版本无法正确报错的问题。([#41335](https://github.com/PaddlePaddle/Paddle/pull/41335)) 

- 修复 DCU 设备上回收显存的 callback 未及时触发导致显存 OOM 的问题。([#40445](https://github.com/PaddlePaddle/Paddle/pull/40445)) 

- 修复 `setitem` 索引赋值反向梯度传递异常以及动态图部分场景下 inplace 逻辑处理异常的问题。 ([#37023](https://github.com/PaddlePaddle/Paddle/pull/37023), [#38298](https://github.com/PaddlePaddle/Paddle/pull/38298)) 

- 修复动转静下 Tensor array 使用 Slice 索引异常的问题。([#39251](https://github.com/PaddlePaddle/Paddle/pull/39251)) 

- 修复 `paddle.Tensor.register_hook` 接口使用时临时变量未析构，从而导致内存或显存泄漏的问题。([#40716](https://github.com/PaddlePaddle/Paddle/pull/40716))

- 修复 `Tensor.getitem` 当索引是全为 False 的 bool Tensor 时无法取值的问题。([#41297](https://github.com/PaddlePaddle/Paddle/pull/41297)) 

- 修复 `Tensor.getitem` 当索引是 bool scalar Tensor 时无法取值的问题。([#40829](https://github.com/PaddlePaddle/Paddle/pull/40829)) 

- 修复 `paddle.index_select` 在 index 为 0-shape Tensor 时报错的问题。([#41383](https://github.com/PaddlePaddle/Paddle/pull/41383)) 

- 修复 `paddle.index_select`，`paddle.index_sample` 申请的 GPU 线程数超过有限机器资源时报错的问题。([#41127](https://github.com/PaddlePaddle/Paddle/pull/41127), [#37816](https://github.com/PaddlePaddle/Paddle/pull/37816), [#39736](https://github.com/PaddlePaddle/Paddle/pull/39736), [#41563](https://github.com/PaddlePaddle/Paddle/pull/41563)) 

- 修复 ReduceConfig、elemwise_grad、gather、gather_nd、scatter ops 申请 GPU 线程数超过有限机器资源时报错的问题。([#40813](https://github.com/PaddlePaddle/Paddle/pull/40813), [#41127](https://github.com/PaddlePaddle/Paddle/pull/41127)) 

- 修复 Kernel Primitive API 中 ReadData，ReadDataBc，ReadDataReduce 在 NX != 1 时访存越界的问题。([#36373](https://github.com/PaddlePaddle/Paddle/pull/36373))

- 修复 IndexRandom 数据类型错误导致数据溢出计算结果异常的问题。([#39867](https://github.com/PaddlePaddle/Paddle/pull/39867), [#39891](https://github.com/PaddlePaddle/Paddle/pull/39891))

- 修复 reduce op 在 reduce_num = 1 计算结果返回错误的问题。([#38771](https://github.com/PaddlePaddle/Paddle/pull/38771))

- 修复 reduce op 在 HIP 环境下 reduce 中间维度出现访存越界的问题。([#41273](https://github.com/PaddlePaddle/Paddle/pull/41273))

- 修复 matmul op 两个 FP16 一维向量计算时 Kernel 无法正常释放的问题。

- 修复部分算子在 CUDA 上因整型计算溢出导致的问题，包括：bernoulli、gaussian_random、gumbel_softmax、multinomial、truncated_gaussian_random、uniform_random_inplace、uniform_random ops。 ([#37670](https://github.com/PaddlePaddle/Paddle/pull/37670))

- 修复 `paddle.nn.Sequential` 在 for 循环遍历 sublayers 时会报 KeyError 错误的 bug。([#39372](https://github.com/PaddlePaddle/Paddle/pull/39372))

- 修复 `paddle.nn.functional.unfold` 在静态图下编译时检查 shape 错误的 bug。([#38907](https://github.com/PaddlePaddle/Paddle/pull/38907), [#38819](https://github.com/PaddlePaddle/Paddle/pull/38819)) 

- 修复静态图使用 dropout 时如果指定了 `axis` 后会报错的问题。([#37223](https://github.com/PaddlePaddle/Paddle/pull/37223))

- 迁移 `paddle.nn.MultiHeadAttention`中 matmul 算子到 matmul_v2 算子。([#36222](https://github.com/PaddlePaddle/Paddle/pull/36222))

- 修复 `paddle.nn.functional.label_smooth`在输入为空 Tensor 时抛出 FPE 的问题。([#35861](https://github.com/PaddlePaddle/Paddle/pull/35861)）

- 修复 reshape op 空 Tensor 形变问题， 支持将空 Tensor rehape 成[-1]。 ([#36087](https://github.com/PaddlePaddle/Paddle/pull/36087)) 

- 修复 `fill_diagonal`参数 offset 非零时会造成修改值跨行问题。([#36212](https://github.com/PaddlePaddle/Paddle/pull/36212))

- 修改动态图模式下 range op 返回 stop gradient 设置成 True。([#37486](https://github.com/PaddlePaddle/Paddle/pull/37486)) 

- 修复 Lamb 优化器当 Beta1Pow 和 Beta2Pow 在 GPU 上时更新错误的 bug。([#38518](https://github.com/PaddlePaddle/Paddle/pull/38518))

- 修复 conv2d 算子 FLAGS_cudnn_deterministic 设置不生效的问题。([#37173](https://github.com/PaddlePaddle/Paddle/pull/37173)) 

- 修复因早期版本的 cufft 没有定义 CUFFT_VERSION 引发的问题。([#37312](https://github.com/PaddlePaddle/Paddle/pull/37312)) 

- 修复 `paddle.ifftshit` , `paddle.fftshift` 计算错误问题。([#36834](https://github.com/PaddlePaddle/Paddle/pull/36834), [#36748](https://github.com/PaddlePaddle/Paddle/pull/36748)) 

- 修复 `paddle.fft` 系列 API 中的 `axis` 计算错误。 ([#36321](https://github.com/PaddlePaddle/Paddle/pull/36321)) 

#### IR(Intermediate Representation)

- 动态图转静态图
  
  - 修复 `tensor_array` 搭配控制流使用时，在反向梯度累加时存在的类型推导错误问题。([#39585](https://github.com/PaddlePaddle/Paddle/pull/39585), [#39689](https://github.com/PaddlePaddle/Paddle/pull/39689)) 
  
  - 修复动转静 AMP 训练时参数梯度类型未被正确设置的问题。([#40938](https://github.com/PaddlePaddle/Paddle/pull/40938)) 
  
  - 修复代码中存在错位注释时，动转静代码解析报错的问题。([#39035](https://github.com/PaddlePaddle/Paddle/pull/39035), [#38003](https://github.com/PaddlePaddle/Paddle/pull/38003)) 
  
  - 修复动转静代码中调用非 forward 函数时，Tensor 未被正确转化为 Variable 的问题。([#37296](https://github.com/PaddlePaddle/Paddle/pull/37296), [#38540](https://github.com/PaddlePaddle/Paddle/pull/38540)) 
  
  - 修复动转静代码转写时 `paddle` 被错误地作为变量传递的问题。([#37999](https://github.com/PaddlePaddle/Paddle/pull/37999)) 
  
  - 修复模型动转静后调用 `paddle.flops` 时模型参数统计错误的问题。([#36852](https://github.com/PaddlePaddle/Paddle/pull/36852)) 
  
  - 修复使用 `paddle.jit.save/load` 接口加载模型后，在 train 模式和 no_grad 上下文中，显存会一直增长的问题。([#36434](https://github.com/PaddlePaddle/Paddle/pull/36434)) 
  
  - 添加在 convert_call 对 generator function 转换时的警告。([#35369](https://github.com/PaddlePaddle/Paddle/pull/35369)) 
  
  - 修复 run_program op 依赖分析的问题。 ([#38470](https://github.com/PaddlePaddle/Paddle/pull/38470)) 
  
  - 修复控制流 For 中返回单值时代码转换的问题。([#40683](https://github.com/PaddlePaddle/Paddle/pull/40683)) 
  
  - 修复控制流 cond 的输入包含 LoDTensorArray 时，生成反向 op 会报错的问题。([#39585](https://github.com/PaddlePaddle/Paddle/pull/39585)) 

#### 分布式训练

- 分布式训练基础功能
  
  - 修复分布式多机训练时，端口报错的问题。([#37274](https://github.com/PaddlePaddle/Paddle/pull/37274)) 
  
  - 修复 brpc 编译依赖问题。([#37064](https://github.com/PaddlePaddle/Paddle/pull/37064)) 
  
  - 修复 Fleet 启动时，由于 tcp 自连接产生的端口被占用的问题。([#38174](https://github.com/PaddlePaddle/Paddle/pull/38174)) 
  
  - 修复数据并行下，由于 FP16 参数在多卡下初始化不一致，导致精度下降的问题。([#38838](https://github.com/PaddlePaddle/Paddle/pull/38838), [#38563](https://github.com/PaddlePaddle/Paddle/pull/38563), [#38405](https://github.com/PaddlePaddle/Paddle/pull/38405))
  
  - 修复数据并行下，由于 FP16 梯度同步时，没有除以卡数，导致精度下降的问题。([#38378](https://github.com/PaddlePaddle/Paddle/pull/38378))

- 动态图混合并行
  
  - 修复在混合并行下，通过使用新 update 接口，FP16 模式不更新参数的问题。([#36017](https://github.com/PaddlePaddle/Paddle/pull/36017))

- 静态图混合并行
  
  - 修复分布式 dp 模式下 grad merge 与 ClipGradientByGlobalNorm 不兼容的问题。([#36334](https://github.com/PaddlePaddle/Paddle/pull/36334)) 
  
  - 修复混合并行下，张量模型并行的非分布式参数在初始化阶段未被广播，导致各卡非分布式参数不一致的问题。([#36186](https://github.com/PaddlePaddle/Paddle/pull/36186)) 
  
  - 修复 sharding 开启 offload 时，sharding 的 save_persistables 接口未保存 FP16 参数和 offload 持久化变量的问题。([#40477](https://github.com/PaddlePaddle/Paddle/pull/40477)) 
  
  - 修复开启 sharding 训练时，ema 参数在非0号卡上无法保存的问题。([#39860](https://github.com/PaddlePaddle/Paddle/pull/39860))
  
  - 修复 FC 按照列切分梯度计算错误的问题。([#38724](https://github.com/PaddlePaddle/Paddle/pull/38724))
  
  - 修复 DistributedStrategy 设置为 without_graph_optimizer 时和 rnn 一起使用报错的问题。 ([#36176](https://github.com/PaddlePaddle/Paddle/pull/36176)) 

- GPUPS 参数服务器训练
  
  - 修复 GPUPS 宏定义触发 CPU 分支编译问题。([#37248](https://github.com/PaddlePaddle/Paddle/pull/37248))
  
  - 修复 GPUPS 流水线训练时在保存 delta 和 pullsparse 并发时引发的偶发报错问题。([#37233](https://github.com/PaddlePaddle/Paddle/pull/37233))
  
  - 修复 HDFSClient 查询目录未返回全路径，引发下载报错问题。 ([#36590](https://github.com/PaddlePaddle/Paddle/pull/36590))
  
  - 修复 GPUPS 流水线训练时拉取老参数问题。([#36512](https://github.com/PaddlePaddle/Paddle/pull/36512)) 
  
  - 修复 GPUPS 多流 allocation 问题。([#37476](https://github.com/PaddlePaddle/Paddle/pull/37476))
  
  - 修复 GPUPS pybind 出 core 的问题。([#37287](https://github.com/PaddlePaddle/Paddle/pull/37287))

#### 其他

- 修复动态图量化训练保存模型时 clip_extra 的问题。([#38323](https://github.com/PaddlePaddle/Paddle/pull/38323))

- 修复动态图量化训练 abs_max scale 初始化的问题。([#39307](https://github.com/PaddlePaddle/Paddle/pull/39307))

- 修复动态图量化训练保存模型节点异常的问题。([#38102](https://github.com/PaddlePaddle/Paddle/pull/38102), [#38012](https://github.com/PaddlePaddle/Paddle/pull/38012))

- 修复离线量化 flatten op 输出错误问题。([#37722](https://github.com/PaddlePaddle/Paddle/pull/37722)) 

- 修复了反量化 matmul op 时，维度对不上的问题。([#36982](https://github.com/PaddlePaddle/Paddle/pull/36982))

- 修复了量化无权重的 matmul_v2 时，错误添加量化 op 的问题。([#36593](https://github.com/PaddlePaddle/Paddle/pull/36593))

- 修复 conv op channel wise 量化在保存模型时 quant_axis 属性保存错误。([#39054](https://github.com/PaddlePaddle/Paddle/pull/39054)) 

- 修复 ChannelWise 量化训练速度慢的问题。([#40772](https://github.com/PaddlePaddle/Paddle/pull/40772))

- 修复量化训练初始化为0的 Tensor 出 NAN 的问题。([#36762](https://github.com/PaddlePaddle/Paddle/pull/36762))

- 修复多线程场景下混合精度 amp_level 设置错误问题。([#39198](https://github.com/PaddlePaddle/Paddle/pull/39198)) 

- 修复混合精度训练与 PyLayer，Recompute 等一起使用时，PyLayer 和 Recompute 中未正确设置混合精度的问题。([#39950](https://github.com/PaddlePaddle/Paddle/pull/39950), [#40042](https://github.com/PaddlePaddle/Paddle/pull/40042)) 

- 修复了 Mac 下编译自定义算子时 `D_GLIBCXX_USE_CXX11_ABI` 未生效的问题。([#37878](https://github.com/PaddlePaddle/Paddle/pull/37878)) 

- 修复 initializer 相关 API 在 block=None 时动静行为不统一的问题。([#37827](https://github.com/PaddlePaddle/Paddle/pull/37827)) 

- 修复 python3.6 环境下没有 fluid 模块的 bug。([#35862](https://github.com/PaddlePaddle/Paddle/pull/35862)) 

- 修复优化器 `paddle.optimizer.Adamw` 错误调用 adam op 的 bug。([#36028](https://github.com/PaddlePaddle/Paddle/pull/36028)) 

- 修复 multi tensor 策略下 `paddle.optimizer.Momentum` 优化器参数 `regularizer` 属性为 None 时的逻辑错误。([#38344](https://github.com/PaddlePaddle/Paddle/pull/38344)) 

- 修复 multi tensor 策略下 `paddle.optimizer.Momentum`、`paddle.optimizer.Adam` 优化器会对 `multi_precision` 属性进行修改的错误。([#38991](https://github.com/PaddlePaddle/Paddle/pull/38991)) 

- 修复最终态 API amp 与 optional 类型 Tensor 组合使用的代码编译错误。([#40980](https://github.com/PaddlePaddle/Paddle/pull/40980)) 

- 修复 paddle+lite+xpu 预测库调用 lite CPU 预测时会报错的 bug，修复 paddle+lite(without NNAdapter) 编译时会报错的 bug。 ([#37449](https://github.com/PaddlePaddle/Paddle/pull/37449)) 

- 修复 Debug 编译模式下 LoDTensorArray 因 Pybind11 绑定不一致导致 crash 的 bug。([#37954](https://github.com/PaddlePaddle/Paddle/pull/37954)) 

- 修复 shape 参数为 Tensor 和 int 构成列表的极端情况下，无法正确构建 Tensor 的 bug。([#38284](https://github.com/PaddlePaddle/Paddle/pull/38284)) 

- 修复 `paddle.optimizer.AdamW` API 兼容性问题。([#37905](https://github.com/PaddlePaddle/Paddle/pull/37905)) 

- 修复 _InstanceNormBase 中 extra_repr 的返回错误。([#38537](https://github.com/PaddlePaddle/Paddle/pull/38537)) 

- 修复联编开启 -DWITH_DISTRIBUTED 生成 Paddle Inference 缺少符号 `paddle::distributed::TensorTable` 的问题。 ([#41128](https://github.com/PaddlePaddle/Paddle/pull/41128)) 

- matmul_v2 op 新增 shape check，在 shape 中存在0值进行信息报错。 ([#35791](https://github.com/PaddlePaddle/Paddle/pull/35791)) 

- 修复动态图 recompute 对于没有梯度输入提示信息反复打印，改成用 warning 只打印一次的方式。([#38293](https://github.com/PaddlePaddle/Paddle/pull/38293)) 

- 修复 gelu op 在视觉模型中训练后期在验证集上精度低的问题。([#38450](https://github.com/PaddlePaddle/Paddle/pull/38450)) 

- 修复 adamw op 在数值计算上误差问题。([#37746](https://github.com/PaddlePaddle/Paddle/pull/37746)) 

- 补充 sparse_momentum `_C_ops` 接口 MasterParam 和 MasterParamOut 参数。([#39969](https://github.com/PaddlePaddle/Paddle/pull/39969)) 

- 修复 python3.6 环境下没有 `distributed` 模块的 bug。([#35848](https://github.com/PaddlePaddle/Paddle/pull/35848)) 

- 修复 eigh 单元测试数据初始化问题。([#39568](https://github.com/PaddlePaddle/Paddle/pull/39568)) 

- 修复 eigvalsh 单元测试数据初始化问题。([#39841](https://github.com/PaddlePaddle/Paddle/pull/39841)) 

- 修复 segment op 在 V100上寄存器使用过多导致不能正常运行的问题。([#38113](https://github.com/PaddlePaddle/Paddle/pull/38113)) 

- 修复 conv 相关算子稀疏化维度错误的问题。([#36054](https://github.com/PaddlePaddle/Paddle/pull/36054)) 

- 提供自动稀疏训练（Automatic SParsity）静态图相关功能 Alias 至 `Paddle.static.sparsity`。([#36525](https://github.com/PaddlePaddle/Paddle/pull/36525)) 

- 修复 divide op 整数除法还是整数的 bug。([#40890](https://github.com/PaddlePaddle/Paddle/pull/40890)) 

- 修复 `paddle.multiplex` 候选 Tensor 大小为0崩溃问题。([#34972](https://github.com/PaddlePaddle/Paddle/pull/34972)) 

- 修复 `paddle.kl_div` 参数 `reduction` 给定情况下速度异常的问题。([#37283](https://github.com/PaddlePaddle/Paddle/pull/37283)) 

- 修复 Cifar 数据集加载 data source 无序的问题。 ([#37272](https://github.com/PaddlePaddle/Paddle/pull/37272)) 

- 修复 ProgressBar 类中 loss 从 uint16 到 float 的转换。([#39231](https://github.com/PaddlePaddle/Paddle/pull/39231)) 

- 修复 ShareBufferWith 共享数据类型的问题。([#37464](https://github.com/PaddlePaddle/Paddle/pull/37464), [#37247](https://github.com/PaddlePaddle/Paddle/pull/37247)) 

- 修复 `paddle.io.DataLoader` 使用 IterableDataset 并且 num_workers>0 时的性能问题。([#40541](https://github.com/PaddlePaddle/Paddle/pull/40541)) 

- 修复 `paddle.vision.ops.yolo_loss` 动态图返回值不全的问题。([#40185](https://github.com/PaddlePaddle/Paddle/pull/40185)) 

- 移出 `paddle.io.BatchSampler` 对输入参数 dataset 需要是 `paddle.io.Dataset` 类型的限制，扩大对用户自定义数据集的支持。([#40184](https://github.com/PaddlePaddle/Paddle/pull/40184)) 

- 修复 `paddle.summary` 报错op_flops不存在的问题。([#36489](https://github.com/PaddlePaddle/Paddle/pull/36489)) 

- 修复 lars_momentum op 在 lars_weight_decay=0 时公式错误的问题。([#40892](https://github.com/PaddlePaddle/Paddle/pull/40892))

- 修复 optimize-offload 无法保存 presistable var 的问题。([#36433](https://github.com/PaddlePaddle/Paddle/pull/36433))

- 修复 optimizer-offload 不支持 adamw op type 的问题。 ([#36432](https://github.com/PaddlePaddle/Paddle/pull/36432))

- 修复多线程场景下，Tracer 中 enable_program_desc_tracing_数据不安全的问题。([#39776](https://github.com/PaddlePaddle/Paddle/pull/39776)) 

- 修复模型读取时模型档案大小未初始化的问题。([#40518](https://github.com/PaddlePaddle/Paddle/pull/40518)) 

- 修复 Expand op 逻辑 bug，当输入Tensor X 的维度，小于要拓展的 shape 时，可能导致取得 Out.Shape 是错误的。([#38677](https://github.com/PaddlePaddle/Paddle/pull/38677))

- 修复 Expand_As op 只取 y.shape，而没有 Y 变量输入时，导致的动转静报错。([#38677](https://github.com/PaddlePaddle/Paddle/pull/38677))

- 修复 Expand_As op 计算输出 shape 时逻辑的错误。([#38677](https://github.com/PaddlePaddle/Paddle/pull/38677))

- 框架功能修复
  
  - 修复 `core.VarDesc.VarType.STRINGS` 类型的变量获取 `lod_level` 属性报错的问题，并且设置其 `lod_level` 为None。([#39077](https://github.com/PaddlePaddle/Paddle/pull/39077))
  
  - 修复框架功能 `Pylayer` 不支持不同 dtype 的问题。 ([#37974](https://github.com/PaddlePaddle/Paddle/pull/37974))

- API修复
  
  - 修复了学习率衰减 API `paddle.optimizer.lr.PolynomialDecay` 的零除问题。 ([#38782](https://github.com/PaddlePaddle/Paddle/pull/38782)) 
  
  - 修复调用 DisableGlogInfo() 接口后依旧残留部分日志的问题。 ([#36356](https://github.com/PaddlePaddle/Paddle/pull/36356)) 

- 修复 SimpleRNN、GRU和LSTM API CPU训练时多层RNN（dropout 设置为0时）反向计算出错的问题。 ([#37080](https://github.com/PaddlePaddle/Paddle/pull/37080)) 

- 为 cufft 和 hipfft 后端的 fft 添加了 cache。 ([#36646](https://github.com/PaddlePaddle/Paddle/pull/36646)) 

- 使 `paddle.roll` 的 shifts 参数支持传入 Tensor。 ([#36727](https://github.com/PaddlePaddle/Paddle/pull/36727)) 

- 为 fft 添加 onemkl 作为可选的计算后端。 ([#36414](https://github.com/PaddlePaddle/Paddle/pull/36414)) 

## 4. 部署方向（Paddle Inference）

### （1）新增特性

#### 新增API

- 增加 Java API，Java 开发者可以通过简单灵活的接口实现在服务端和云端的高性能推理。([#37162](https://github.com/PaddlePaddle/Paddle/pull/37162))

- 增加 `GetTrtCompileVersion` 和 `GetTrtRuntimeVersion` 接口，用于获取 TensorRT 版本信息。([#36429](https://github.com/PaddlePaddle/Paddle/pull/36429))

- 增加 `ShareExternalData` 接口，避免推理时对输入数据进行内存拷贝。([#39809](https://github.com/PaddlePaddle/Paddle/pull/39809))

#### 新增功能

- 新增 ONNX Runtime 后端支持，当前集成版本只支持 CPU。([#39988](https://github.com/PaddlePaddle/Paddle/pull/39988), [#40561](https://github.com/PaddlePaddle/Paddle/pull/40561))

- 基于 Paddle Lite 子图方式，新增昇腾310推理支持。([#35226](https://github.com/PaddlePaddle/Paddle/pull/35226))

- 新增原生 GPU FP16 推理功能。([#40531](https://github.com/PaddlePaddle/Paddle/pull/40531))

- switch_ir_debug 接口增加 dump 模型的功能。([#36581](https://github.com/PaddlePaddle/Paddle/pull/36581))

- 新增 TensorRT config 的配置接口：`void UpdateConfigInterleaved(paddle_infer::Config* c, bool with_interleaved)`，用于 int8 量化推理中特殊的数据排布。([#38884](https://github.com/PaddlePaddle/Paddle/pull/38884)) 

- log 中增加 TensorRT inspector 输出信息，仅在 TensorRT 8.2及以上版本有效。 ([#38362](https://github.com/PaddlePaddle/Paddle/pull/38362)，[#38200](https://github.com/PaddlePaddle/Paddle/pull/38200))) 

- 增加 TensorRT ASP 稀疏推理支持。([#36413](https://github.com/PaddlePaddle/Paddle/pull/36413))

### （2）底层优化

#### CPU性能优化

- 优化 MKLDNN 的缓存机制。([#38336](https://github.com/PaddlePaddle/Paddle/pull/38336), [#36980](https://github.com/PaddlePaddle/Paddle/pull/36980), [#36695](https://github.com/PaddlePaddle/Paddle/pull/36695)) 

- 新增 matmul_scale_fuse pass。([#37962](https://github.com/PaddlePaddle/Paddle/pull/37962)) 

- 新增 MKLDNN reshape_transpose_matmul_v2_mkldnn_fuse_pass。([#37847](https://github.com/PaddlePaddle/Paddle/pull/37847), [#40948](https://github.com/PaddlePaddle/Paddle/pull/40948)) 

- 新增 MKLDNN conv_hard_sigmoid_mkldnn_fuse_pass。([#36869](https://github.com/PaddlePaddle/Paddle/pull/36869))

- 新增 MKLDNN matmul_v2_transpose_reshape_fuse_pass。([#36481](https://github.com/PaddlePaddle/Paddle/pull/36481))

- 新增 MKLDNN softplus_activation_mkldnn_fuse_pass。([#36657](https://github.com/PaddlePaddle/Paddle/pull/36657))

- 新增 MKLDNN elt_act_mkldnn_fuse_pass。([#36541](https://github.com/PaddlePaddle/Paddle/pull/36541))

- 新增 MKLDNN mish 算子及 conv_mish_mkldnn_fuse_pass。([#38623](https://github.com/PaddlePaddle/Paddle/pull/38623))

#### GPU 性能优化

- 将推理默认的显存分配策略由 `naive_best_fit` 变更为 `auto_growth`，解决部分模型占满 GPU 显存问题。([#41491](https://github.com/PaddlePaddle/Paddle/pull/41491))

- 支持 gelu、FC+gelu ops 使用 TensorRT 推理。([#38399](https://github.com/PaddlePaddle/Paddle/pull/38399))合作团队

- 支持 `deformable_conv` 在静态 shape下使用 TensorRT 推理。([#36612](https://github.com/PaddlePaddle/Paddle/pull/36612) [#36850](https://github.com/PaddlePaddle/Paddle/pull/36850) [#37345](https://github.com/PaddlePaddle/Paddle/pull/37345))

- 支持 nearest_interp_v2 op 使用 TensorRT 推理。([#34126](https://github.com/PaddlePaddle/Paddle/pull/34126))

- 增加 `yolo_box`TensorRT plugin，支持输入参数 `iou_aware` 和 `iou_aware_factor`，使推理计算得到的 IoU 作为置信度的因子。([#34128](https://github.com/PaddlePaddle/Paddle/pull/34128))

- 支持 `elementwise_sub` 和 `elementwise_div` 调用 TensorRT 推理。([#40806](https://github.com/PaddlePaddle/Paddle/pull/40806) [#41253](https://github.com/PaddlePaddle/Paddle/pull/41253))

- 支持 `multiclass_nms3` 使用 TensorRT 推理。([#41181](https://github.com/PaddlePaddle/Paddle/pull/41181) [#41344](https://github.com/PaddlePaddle/Paddle/pull/41344))

- 支持 flatten_contiguous_rang op 使用 TensorRT 推理。([#38922](https://github.com/PaddlePaddle/Paddle/pull/38922)) 

- 支持 `pool2d` 属性 `padding` 的维度为4、`global_pooling` 和 `ceil_mode` 为 True 情况下使用 TensorRT 推理。([#39545](https://github.com/PaddlePaddle/Paddle/pull/39545))

- 支持 batch_norm 和 elementwise_add 为5维时使用 TensorRT 推理。([#36446](https://github.com/PaddlePaddle/Paddle/pull/36446))

- 新增 pool3d 使用 TensorRT 推理。([#36545](https://github.com/PaddlePaddle/Paddle/pull/36545), [#36783](https://github.com/PaddlePaddle/Paddle/pull/36783)) 

- 增加 `reduce` int32 和 float 类型使用 TensorRT 推理，增加 `reduce_mean` GPU 算子 int32、int64 注册。([#39088](https://github.com/PaddlePaddle/Paddle/pull/39088))

- 修改 MatmulV2ToMul pass，修改限定条件（不支持广播）和 op_teller 映射条件。([#36652](https://github.com/PaddlePaddle/Paddle/pull/36652)) 

- 增加 TenorRT plugin 接口 AddPluginV2IOExt 的支持 。([#36493](https://github.com/PaddlePaddle/Paddle/pull/36493)) 

- 增加 roi_align op 中 aligned 属性并支持 TensorRT 推理。([#38905](https://github.com/PaddlePaddle/Paddle/pull/38905))

- 增加 concat 属性 `axis = -1` 时支持 TensorRT 推理。([#39096](https://github.com/PaddlePaddle/Paddle/pull/39096)) 

- 新增 TensorRT plugin ：preln_emb_eltwise_layernorm、 preln_skip_la、rnorm ops， 用于 ERNIE 类模型性能优化。([#39570](https://github.com/PaddlePaddle/Paddle/pull/39570))

- 新增 TensorRT fuse pass：preln_embedding_eltwise_layernorm_fuse_pass, preln_skip_layernorm_fuse_pass，用于 ERNIE 类模型性能优化。([#39508](https://github.com/PaddlePaddle/Paddle/pull/39508))

- 将 matmul 融合相关的 pass 基于不同的后端（GPU、CPU、TensorRT）拆开，支持 FC 权重的转置功能。([#39369](https://github.com/PaddlePaddle/Paddle/pull/39369))

- 量化支持
  
  - `PostTrainingQuantization` API新增支持`paddle.io.DataLoader` 对象或者 `Python Generator`的输入。([#38686](https://github.com/PaddlePaddle/Paddle/pull/38686))
  
  - ERNIE 全量化模型推理支持 interleaved 数据排布。([#39424](https://github.com/PaddlePaddle/Paddle/pull/39424)) 
  
  - 支持 PaddleSlim 新量化模型格式推理。([#41049](https://github.com/PaddlePaddle/Paddle/pull/41049)) 
  
  - 新增 matmul int8 量化的推理 op converter 和 plugin。([#37285](https://github.com/PaddlePaddle/Paddle/pull/37285))
  
  - 新增判断模型所有 op 能否支持 int8 量化的 pass。([#36042](https://github.com/PaddlePaddle/Paddle/pull/36042))
  
  - 支持 multihead attention 非变长分支中 FC 部分的量化推理。([#39660](https://github.com/PaddlePaddle/Paddle/pull/39660)) 

#### 昇腾NPU 相关功能

- - 重构 shape 算子前向计算逻辑，支持在 NPU 上执行。([#39613](https://github.com/PaddlePaddle/Paddle/pull/39613))
  
  - 重构 reshape 算子前向计算逻辑，支持 ShapeTensor 输入。([#38748](https://github.com/PaddlePaddle/Paddle/pull/38748))
  
  - 模型权重加载时精度类型统一。([#39160](https://github.com/PaddlePaddle/Paddle/pull/39160))

### （3）问题修复

#### 框架及API修复

- 修复保存静态图时模型剪裁的问题。([#37579](https://github.com/PaddlePaddle/Paddle/pull/37579))

- C API 增加对的字符串的封装 PD_Cstr，并提供构造和析构的方式，避免用户直接使用 C 运行时库来析构字符串。 ([#38667](https://github.com/PaddlePaddle/Paddle/pull/38667)) 

- 修复预测时内存复用的逻辑问题。([#37324](https://github.com/PaddlePaddle/Paddle/pull/37324))

- 修复多线程下内存复用报错问题。([#37894](https://github.com/PaddlePaddle/Paddle/pull/37894)) 

- 在没有权重文件时，允许传递空字符串进行推理。([#38579](https://github.com/PaddlePaddle/Paddle/pull/38579))

- 修复开启 TensorRT dynamic shape 后不支持 clone 问题。([#38520](https://github.com/PaddlePaddle/Paddle/pull/38520))

- 修复开启 TensorRT dynamic shape 后多线程 clone 报错问题。([#40067](https://github.com/PaddlePaddle/Paddle/pull/40067))

- 修复 TensorRT engine 析构问题。([#35842](https://github.com/PaddlePaddle/Paddle/pull/35842), [#35938](https://github.com/PaddlePaddle/Paddle/pull/35938))

- lite xpu 接口修复无法选择 xpu 卡的问题。([#36610](https://github.com/PaddlePaddle/Paddle/pull/36610)) 

- TensorRT 动态 shape 参数自动生成接口增加文件存在性检查。([#36628](https://github.com/PaddlePaddle/Paddle/pull/36628))

#### 后端能力修复

- 修复预测时 cuDNN 默认算法选择配置，使用非 deterministic 策略。 ([#41491](https://github.com/PaddlePaddle/Paddle/pull/41491))

- 修复 deformable_conv op 在 TensorRT plugin 资源回收处理错误的问题。 ([#38374](https://github.com/PaddlePaddle/Paddle/pull/38374)) 

- 修复 deformable_conv op 在 TensorRT plugin 序列化错误问题。 ([#38057](https://github.com/PaddlePaddle/Paddle/pull/38057)) 

- 适配 TensorRT 8.0 新的构建引擎和系列化 API。 ([#36769](https://github.com/PaddlePaddle/Paddle/pull/36769)) 

- 修复 Flatten2MatmulFusePass、Squeeze2MatmulFusePass、Reshape2MatmulFusePass 没有生效问题。([#37644](https://github.com/PaddlePaddle/Paddle/pull/37644)) 

- 修复 TensorRT 输入数据在上时报错的问题。([#37427](https://github.com/PaddlePaddle/Paddle/pull/37427))

- 增加输入维度错误时的报错信息。([#38962](https://github.com/PaddlePaddle/Paddle/pull/38962))

- 修复 EmbEltwiseLayernorm 输出类型错误的问题。 ([#40015](https://github.com/PaddlePaddle/Paddle/pull/40015))

- 删除 conv_affine_channel_fuse_pass 以及对应的单元测试。([#39817](https://github.com/PaddlePaddle/Paddle/pull/39817))

- 修复 adaptive_pool2d pass 错误替换 pool 属性的问题。([#39600](https://github.com/PaddlePaddle/Paddle/pull/39600))

- 修复 shuffle_channel_detect_pass 错误生成 shuffle_channel op 的问题。([#39242](https://github.com/PaddlePaddle/Paddle/pull/39242))

- 修复 transpose 参数错误。([#39006](https://github.com/PaddlePaddle/Paddle/pull/39006))

- 修复 nearest_interp_v2 输入 scale 维度小于1时崩溃的问题。([#38725](https://github.com/PaddlePaddle/Paddle/pull/38725))

- 修复 prelu 在 dynamic shape 时不支持一维输入的问题。([#39389](https://github.com/PaddlePaddle/Paddle/pull/39389))

- 修复 slice 的 special_slice_plugin 的核函数计算错误的问题。([#39875](https://github.com/PaddlePaddle/Paddle/pull/39875)) 

- 暂时禁用 skip_layernorm 变长下的 int8 分支，防止精度下降。([#39991](https://github.com/PaddlePaddle/Paddle/pull/39991)) 

- 修复关于支持 preln_ernie 模型的一些 bug。([#39733](https://github.com/PaddlePaddle/Paddle/pull/39733))

- 修复 slice 在 ERNIE 中 threads 可能超过限制的 bug，修复 spacial_slice 误触的 bug。([#39096](https://github.com/PaddlePaddle/Paddle/pull/39096)) 

- 修复 elementwise 在维度相同时不支持广播的问题。([#37908](https://github.com/PaddlePaddle/Paddle/pull/37908)) 

- 修复 nearest_interp op 当 align_corners 为 True 时，TensorRT layer 的结果和原生 op 的结果有 diff，底层实现不一样。([#37525](https://github.com/PaddlePaddle/Paddle/pull/37525)) 

- 修复qkv_plugin: 核函数计算错误。([#37096](https://github.com/PaddlePaddle/Paddle/pull/37096)) 

- 修复动态量化的推理 pass 的问题。([#35879](https://github.com/PaddlePaddle/Paddle/pull/35879)) 

- 当 Tensor 请求的内存容量低于已分配的 size 时直接复用。([#37880](https://github.com/PaddlePaddle/Paddle/pull/37880))

- 修复 ERNIE 定长模型开启 TensorRT 出现的 hang 问题。([#37839](https://github.com/PaddlePaddle/Paddle/pull/37839))

- 修复 TensorRT int8 时缺失 dynamic range 信息崩溃问题。([#36900](https://github.com/PaddlePaddle/Paddle/pull/36900))

- 修复 slice 反序列化代码问题。([#36588](https://github.com/PaddlePaddle/Paddle/pull/36588))

- 修复 yolo box 计算公式错误问题。([#36240](https://github.com/PaddlePaddle/Paddle/pull/36240))

- 修复老版本模型在使用新版本 roi_align 时崩溃问题。([#38788](https://github.com/PaddlePaddle/Paddle/pull/38788)) 外部开发者

- 修复 softmax 在 python 和 C++上性能差异较大的问题。([#37130](https://github.com/PaddlePaddle/Paddle/pull/37130)) 

- 修复 matmul 在静态 shape 2维输入和动态 shape 3维输入情况下推理失败问题。([#36849](https://github.com/PaddlePaddle/Paddle/pull/36849))

- 修复 reshape_transpose_matmul_mkldnn_fuse_pass 对 shape 处理不当问题。([#36731](https://github.com/PaddlePaddle/Paddle/pull/36731))

- 修复输入为2维，但 TensorRT 获取到4维的问题。([#36614](https://github.com/PaddlePaddle/Paddle/pull/36614))

- 修复 interpolate_v2 MKLDNN 算子在 scale 属性为空时报错问题。([#36623](https://github.com/PaddlePaddle/Paddle/pull/36623))

- 修复 recurrent 算子在多线程场景性能差问题。([#36052](https://github.com/PaddlePaddle/Paddle/pull/36052))

- 移除 relu、sigmoid、tanh、relu6、batch_norm、clip、concat、gelu、hard_sigmoid、prelu、softmax、split、swish 对 TensorRT 2维输入的限制。([#37097](https://github.com/PaddlePaddle/Paddle/pull/37097))

- 修复 reshape op 使用 TensorRT 推理。([#41090](https://github.com/PaddlePaddle/Paddle/pull/41090))

- 修复 matmul 相关 pass，兼容 matmul_v2。([#36424](https://github.com/PaddlePaddle/Paddle/pull/36424))

- 开启 TensorRT 时，conv2d 算子中 padding 方式支持 VALID 及 SAME 属性。([#38999](https://github.com/PaddlePaddle/Paddle/pull/38999))

- 修复 MKLDNN 多输入算子量化问题。([#39593](https://github.com/PaddlePaddle/Paddle/pull/39593), [#39346](https://github.com/PaddlePaddle/Paddle/pull/39346), [#40717](https://github.com/PaddlePaddle/Paddle/pull/40717)) 

- 修复 MKLDNN 量化场景下 conv+activation 的 scale 错误问题。([#38331](https://github.com/PaddlePaddle/Paddle/pull/38331)) 

- 修复 MKLDNN 无参数算子量化中，根据后续算子量化情况不同需做不同处理的问题。([#39342](https://github.com/PaddlePaddle/Paddle/pull/39342)) 

- 修复 MKLDNN cpu_bfloat16_placement_pass 中的数据类型相关问题。([#38702](https://github.com/PaddlePaddle/Paddle/pull/38702)) 

- 修复 MKLDNN bfloat16 推理中 split 算子执行问题。([#39548](https://github.com/PaddlePaddle/Paddle/pull/39548)) 

- 修复 MKLDNN matmul_v2 算子不支持6维问题。([#36342](https://github.com/PaddlePaddle/Paddle/pull/36342), [#38665](https://github.com/PaddlePaddle/Paddle/pull/38665)) 

- 修复 MKLDNN matmul_v2_transpose_reshape 中的 MKLDNN DeviceContext 错误问题。([#38554](https://github.com/PaddlePaddle/Paddle/pull/38554)) 

- 修复分割模型在 MKLDNN 推理场景计算结果错误问题。([#37310](https://github.com/PaddlePaddle/Paddle/pull/37310)) 

- 修复 MKLDNN bfloat16 placement 算子列表并添加缺失算子。([#36291](https://github.com/PaddlePaddle/Paddle/pull/36291)) 

- 修复 MKLDNN 算子的格式问题，包括: FC、conv_transpose、6维 Tensor 报错问题、conv 对 `NHWC` 输入的输出 format 错误问题。([#38890](https://github.com/PaddlePaddle/Paddle/pull/38890), [#37344](https://github.com/PaddlePaddle/Paddle/pull/37344), [#37175](https://github.com/PaddlePaddle/Paddle/pull/37175), [#38553](https://github.com/PaddlePaddle/Paddle/pull/38553), [#40049](https://github.com/PaddlePaddle/Paddle/pull/40049), [#39097](https://github.com/PaddlePaddle/Paddle/pull/39097)) 

- 修复 MKLDNN 多线程推理场景因 cache 机制报错问题。([#36290](https://github.com/PaddlePaddle/Paddle/pull/36290), [#35884](https://github.com/PaddlePaddle/Paddle/pull/35884)) 

- 修复 MKLDNN 因 matmul 及 FC 引起的量化模型精度异常问题。([#38023](https://github.com/PaddlePaddle/Paddle/pull/38023), [#37618](https://github.com/PaddlePaddle/Paddle/pull/37618)) 

- 修复 MKLDNN 量化转换脚本因 pass 缺少引起的量化模型精度异常问题。([#37619](https://github.com/PaddlePaddle/Paddle/pull/37619), [#40542](https://github.com/PaddlePaddle/Paddle/pull/40542),  
  [#38912](https://github.com/PaddlePaddle/Paddle/pull/38912)) 

- 修复 MKLDNN 开启量 op 因为数据类型不匹配崩溃的问题。([#38133](https://github.com/PaddlePaddle/Paddle/pull/38133))

- 修复 MKLDNN 某些 op 修改 layout 后需要改回原 layout 的问题。([#39422](https://github.com/PaddlePaddle/Paddle/pull/39422))

- 修复针对昇腾910推理场景下，由于未释放 GIL 锁，导致与昇腾软件栈冲突，python API 下报错的问题。 ([#38605](https://github.com/PaddlePaddle/Paddle/pull/38605)) 

## 5. 环境适配

### 编译安装

- 从2.3.0-rc0版本开始，飞桨对框架支持的 GPU 架构种类进行了调整和升级。(更多请参考: [飞桨支持的 GPU 架构](https://www.paddlepaddle.org.cn/documentation/docs/zh/2.3rc/install/Tables.html#gpu))

备注：

- PIP 源安装是指用 `pip install paddlepaddle` 或 `pip install paddlepaddle-gpu`从 PIP 官网下载安装包及依赖库的安装方式，支持架构种类少，安装包更轻量，下载源来自国外（相比bos源支持架构种类精简，安装包更轻量，只提供一种 CUDA 版本的安装包）。
  
  - 2.3版本之前，飞桨 PIP 源安装包（CUDA10.2）支持的 GPU 架构为：3.5, 5.0, 5.2, 6.0, 6.1, 7.0, 7.5。
  
  - 2.3版本之后，飞桨 PIP 源安装包（CUDA11.0）支持的 GPU 架构为：6.0, 6.1, 7.0, 7.5, 8.0

- 飞桨官网 bos 源是指从飞桨官网下载安装包及依赖库的安装方式，支持的 GPU 架构更多，下载源来自国内，速度较快。（相比PIP源支持架构种类多，提供多个 CUDA 版本的安装包）：
  
  - 2.3版本之前，飞桨官网 bos 源安装包支持的 GPU 架构：
    
    - CUDA10 : 3.5, 5.0, 5.2, 6.0, 6.1, 7.0, 7.5；
    
    - CUDA11 : 5.2，6.0，6.1，7.0，7.5，8.0。
  
  - 2.3版本之后，飞桨官网 bos 源安装包支持的 GPU 架构
    
    - CUDA10 : 3.5, 5.0, 5.2, 6.0, 6.1, 7.0, 7.5；
    
    - CUDA11 : 3.5, 5.0, 6.0, 6.1, 7.0, 7.5, 8.0。

- Windows 平台支持 Visual Studio 2019 编译。 ([#38719](https://github.com/PaddlePaddle/Paddle/pull/38719)) 

- 消除 Windows 平台编译时出现的各种 warning。 ([#38034](https://github.com/PaddlePaddle/Paddle/pull/38034), [#37890](https://github.com/PaddlePaddle/Paddle/pull/37890), [#37442](https://github.com/PaddlePaddle/Paddle/pull/37442), [#37439](https://github.com/PaddlePaddle/Paddle/pull/37439), [#36857](https://github.com/PaddlePaddle/Paddle/pull/36857)) 

- 修复底层数据结构升级引入的 jetson 编译问题。 ([#39669](https://github.com/PaddlePaddle/Paddle/pull/39669), [#39441](https://github.com/PaddlePaddle/Paddle/pull/39441))

### 新硬件适配

- 自定义新硬件接入：提供一种插件式扩展 PaddlePaddle 硬件后端的方式。通过该功能，开发者无需为特定硬件修改 PaddlePaddle 代码，只需实现标准接口，并编译成动态链接库，则可作为插件供 PaddlePaddle 调用。降低为 PaddlePaddle 添加新硬件后端的开发难度。当前支持自定义 Runtime 接入和自定义 Kernel 接入。

- 华为 NPU 芯片（Ascend910）训练/推理支持，支持ResNet50、YoloV3、BERT、Transformer等多个模型，支持静态图与混合精度训练，支持单卡、单机、多机分布式训练。

- Graphcore IPU芯片（包括IPU Mk2 GC200 和 Bow IPU）训练/推理支持，支持ResNet50、BERT等模型，支持静态图训练，支持单芯片、单机、多机分布式训练。

- 寒武纪MLU芯片（MLU370x4）训练/推理支持，支持ResNet50等模型，支持静态图+动态图训练，支持混合精度训练，支持单卡、单机、多机分布式训练。

- 昆仑芯2代芯片（昆仑芯 AI加速卡 R200、R300）训练/推理支持，支持ResNet50、YoloV3、OCR-DB、SSD、MobilnetV3、UNet、BERT、Transformer、GPT-2、Wide&Deep、DeepFM，支持静态图+动态图训练，支持混合精度训练，支持单机单卡、单机多卡训练。

## Thanks to our Contributors

This release contains contributions from the project core team as well as :

Adam Osewski, Allen Guo, arlesniak, chenenquan, chenyanlann, fengkuangxiaxia, fuqianya, fwenguang, guguguzi, helen88, houj04, Jacek Czaja, jakpiase, jianghaicheng, joanna.wozna.intel, joeqiao12, Leo Chen, Leo Guo, Li-fAngyU, lidanqing, Liyulingyue, Matsumoto GAO, maxhuiy, Ming-Xu Huang, Nyakku Shigure, piotrekobi, piotrekobiIntel, QingshuChen, qipengh, Skr Bang, Sylwester Fraczek, Sławomir Siwek, taixiurong, tanzhipeng, Tomasz Socha, TTerror, Webbley, yaozhixin, ykkk2333, yujun, Zhangjingyu06, zhangxiaoci, zhangyikun02, zhangyk0314, zlsh80826, zn, Zuza

[English](https://github.com/zerollzeng/tiny-tensorrt/blob/master/README.md) | 中文简体
# tiny-tensorrt
一个非常高效易用的nvidia TensorRT封装,支持c++,python调用,支持caffe,onnx,tensorflow模型.

# 注意事项
TensorRT更新了6.0的版本,api做了许多修改,所以我也升级tiny-tensorrt到6.0版本,5.x的版本在trt-5.1.5.0分支

但是请注意新的特性和更新不会同步到5.x分支,所以我建议使用最新的版本

# 里程碑
- [x] 自定义插件教程和非常详细的示例代码! 12月13日之前可以更新完 :fire::fire::fire:
- [x] tensorflow模型支持
- [x] 自定义onnx模型输出节点 ---2019.10.18
- [x] 升级到TensorRT 6.0.1.5 --- 2019.9.29
- [ ] 实现更多的层(有需要请给我提issue阿喂,但是我希望是通用的层) --working on
- [x] caffe模型支持
- [x] PRELU支持
- [x] upsample支持
- [x] 引擎序列化
- [x] caffe model int8支持
- [x] onnx支持
- [x] python api支持
- [x] 在p4显卡上测试
- [x] 自定义使用显卡

# 系统需求
TensorRT 6.x 版本和 cuda 10.0+

如果要使用python api, 那么还需要安装python2/3 和 numpy

# 安装
首先确保你已经安装了上述依赖, 如果你熟悉docker的话,也可以用[官方docker镜像](https://ngc.nvidia.com/catalog/containers/nvidia:tensorrt)
```bash
# clone project and submodule
git clone --recurse-submodules -j8 https://github.com/zerollzeng/tiny-tensorrt.git

cd tiny-tensorrt

mkdir build && cd build && cmake .. && make
```
then you can intergrate it into your own project with libtinytrt.so and Trt.h, for python module, you get pytrt.so
然后你就可以将它集成进你自己的项目里面,只需要libtinytrt.so和Trt.h, 如果要用python api, 使用pytrt.so

# 使用方法
c++
```c++
#include "Trt.h"

Trt trt;
// create engine and running context, note that engine file is device specific, so don't copy engine file to new device, it may cause crash
trt.CreateEngine("path/to/sample.prototxt",
                 "path/to/sample.caffemodel",
                 "path/to/engineFile", // since build engine is time consuming,so save we can serialize engine to file, it's much more faster
                 "outputblob",
                 calibratorData,
                 maxBatchSize,
                 runMode);
// trt.CreateEngine(onnxModel,engineFile,maxBatchSize); // for onnx model

// you might need to do some pre-processing in input such as normalization, it depends on your model.
trt.DataTransfer(input,0,True); // 0 for input index, you can get it from CreateEngine phase log output, True for copy input date to gpu

//run model, it will read your input and run inference. and generate output.
trt.Forward();

//  get output.
trt.DataTransfer(output, outputIndex, False) // you can get outputIndex in CreateEngine phase
// them you can do post processing in output
```

python
```python
import sys
sys.path.append("path/to/where_pytrt.so_located/")
import pytrt

trt = pytrt.Trt()
trt.CreateEngine(prototxt, caffemodel, engineFile, outputBlobName, calibratorData, maxBatchSize, mode)
# trt.CreateEngine(onnxModel, engineFile, maxBatchSize)
# see c++ CreateEngine

trt.DoInference(input_numpy_array) # slightly different from c++
output_numpy_array = trt.GetOutput(outputIndex)
# post processing
```

also see [tensorrt-zoo](https://github.com/zerollzeng/tensorrt-zoo), it implement some common computer vision model with tiny tensor_rt, it has serveral good samples

# 支持的额外层
- 自定义尺度upsample,在yolov3上测试
- yolo-det, 就是yolov3的最后一层,将三个尺度的输出集合起来产生检测结果
- PRELU, 在openpose和mtcnn上做了测试

# 致谢
这个项目最初是因为看到了[lewes6369/tensorRTWrapper](https://github.com/lewes6369/tensorRTWrapper) 和 [lewes6369/TensorRT-Yolov3](https://github.com/lewes6369/TensorRT-Yolov3),所以打算做一个更好的.

然后我使用了[spdlog](https://github.com/gabime/spdlog),可以有比较好看的log信息,看起来更清晰一点

使用了pybind11来做python api

# 版权
这个项目包含了一些第三方模块,对于它们需要遵守他们的版权要求

对于我写的那一部分,你可以做任何你想做的事
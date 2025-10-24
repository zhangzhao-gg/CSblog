我需要量化一个音频回声擦除模型，加快推理速度，模型本身格式是ONNX，我采用ONNXRuntime进行动态量化。
代码如下：
```
from onnxruntime.quantization import quantize_dynamic, QuantType

model_input = "./pretrained_models/dtln_aec_128_2.onnx"
model_output = "./pretrained_models/dtln_aec_128_int8_2.onnx"

quantize_dynamic(
    model_input=model_input,
    model_output=model_output,
    op_types_to_quantize=[“Conv”,"Gemm", "MatMul", "Add", "Mul"],
    per_channel=True,
    weight_type=QuantType.QUInt8,
    extra_options={
            "ActivationSymmetric": False,
            "WeightSymmetric": True,
            "EnableSubgraph": True,
            "ForceQuantizeNoInputCheck": False,
            "MatMulConstBOnly": True
        }
)


print("✅ INT8 动态量化完成：", model_output)
```

量化能够成功，但是在推理的时候遇到如下报错：
`NOT_IMPLEMENTED : Could not find an implementation for ConvInteger(10) node with name 'Conv_0_quant'`

翻译过来就是：在推理的时候没有找到ConvInteger算子。
我翻了官方网站的说法ConvInteger算子是动态量化后的Conv算子。这就很奇怪，怎么可能官方不支持动态量化后的卷子算子呢，卷积算子是个很常见的算子啊。
下面是踩坑记录：
踩坑1:
找了csdn，唯一一个遇到跟我一样问题的人，他把int8量化改成了Uint8量化就推理的时候不报错了，`weight_type=QuantType.QUInt8, ` 但我发现这么做会导致量化后的推理速度甚至慢于量化前推理速度

下面公布正确答案：
正确答案：[在这里](https://github.com/microsoft/onnxruntime/issues/15888) 官方就是不支持动态量化算子！ 
那么按照这个issue里面来看，方法就是在量化的时候过滤掉卷积算子
`op_types_to_quantize=["Gemm", "MatMul", "Add", "Mul"],`
真坑啊，从22年到现在onnxruntime最新版也有这个问题。。。

那卷积算子没有被量化会影响推理速度，有没有办法。
按照官方的说法，他们更支持对卷积为主的模型进行静态量化，我没尝试过，给你链接：
https://onnxruntime.ai/docs/performance/model-optimizations/quantization.html#method-selection

<img width="838" height="312" alt="Image" src="https://github.com/user-attachments/assets/f4125302-a65f-4196-b669-180bf244122e" />

下载地址：https://modelscope.cn/models/cpatonn-mirror/Qwen3-Omni-30B-A3B-Instruct-AWQ-8bit
注意一下这个界面里面的下载命令是错的，还用的原始模型的下载命令

正确的下载命令：
```
pip install -U modelscope
modelscope download --model cpatonn-mirror/Qwen3-Omni-30B-A3B-Instruct-AWQ-8bit --local_dir ./Qwen3-Omni-30B-A3B-Instruct-8bit

```
下载下来大概有几十个G。
下载完毕后，配置VLLM环境，VMML不能用官方的，因为模型太新了，官方的暂时不支持该模型。我们用作者提供的操作：
```
git clone -b qwen3_omni https://github.com/wangxiongts/vllm.git
cd vllm
pip install -r requirements/build.txt
pip install -r requirements/cuda.txt
export VLLM_PRECOMPILED_WHEEL_LOCATION=https://wheels.vllm.ai/a5dd03c1ebc5e4f56f3c9d3dc0436e9c582c978f/vllm-0.9.2-cp38-abi3-manylinux1_x86_64.whl
VLLM_USE_PRECOMPILED=1 pip install -e . -v --no-build-isolation
# If you meet an "Undefined symbol" error while using VLLM_USE_PRECOMPILED=1, please use "pip install -e . -v" to build from source.
# Install the Transformers
pip install git+https://github.com/huggingface/transformers
pip install accelerate
pip install qwen-omni-utils -U
pip install -U flash-attn --no-build-isolation

```


**坑1:**
其中
`git clone -b qwen3_omni https://github.com/wangxiongts/vllm.git`
可能会遇到网络不好的情况下载很慢，那么就可以考虑手动下载zip到本地，然后再上传到服务器。若这样做。
那么会有一个坑就是在执行`VLLM_USE_PRECOMPILED=1 pip install -e . -v --no-build-isolation`的时候会显示找不到版本号，因为直接从github下载zip文件时候是不带版本的。
解决方案：输入`export SETUPTOOLS_SCM_PRETEND_VERSION=0.9.2`
解决原理：假装我们有个版本，就可以继续顺利编译安装了

**坑2:**
其中
`pip install git+https://github.com/huggingface/transformers`可能会网络卡，换成下面的
`pip install git+https://gitee.com/mirrors/huggingface-transformers.git`

**坑3:**
pip install -U flash-attn --no-build-isolation 会卡住
自己去下载对应的whl包，[地址在这里](https://github.com/mjun0812/flash-attention-prebuild-wheels/releases)
下载好之后上传到服务器内，然后`pip install flash_attn-2.8.3+cu124torch2.5-cp310-cp310-linux_x86_64.whl`
这个`flash_attn-2.8.3+cu124torch2.5-cp310-cp310-linux_x86_64.whl`是我下载的，你要是不确定也可以用我这个

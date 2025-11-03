魔塔社区的链接在这里
https://modelscope.cn/models/Qwen/Qwen3-Omni-30B-A3B-Instruct/summary
全程科学上网，且重新开一个纯净的新环境，在新环境里配环境。

1安装transformers，需要源码下载
`pip install git+https://github.com/huggingface/transformers`
2`pip install accelerate`
3`pip install qwen-omni-utils -U`
4
`pip install -U flash-attn --no-build-isolation`
这一步就有问题，如果不开科学上网，他会卡在编译过程非常慢。
autoDL支持科学上网：
```
source /etc/network_turbo
python -m pip install ninja -i https://pypi.tuna.tsinghua.edu.cn/simple
```
很快编译完成啦
然后运行，报错
```
(qwen) root@autodl-container-0a4d4f962f-c0ee9df0:~/autodl-tmp# python omni.py
Unrecognized keys in `rope_parameters` for 'rope_type'='default': {'mrope_section', 'mrope_interleaved', 'interleaved'}
Unrecognized keys in `rope_parameters` for 'rope_type'='default': {'mrope_section', 'interleaved'}
Traceback (most recent call last):
  File "/root/autodl-tmp/omni.py", line 9, in <module>
    model = Qwen3OmniMoeForConditionalGeneration.from_pretrained(
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/modelscope/utils/hf_util/patcher.py", line 285, in from_pretrained
    module_obj = module_class.from_pretrained(
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/modeling_utils.py", line 270, in _wrapper
    return func(*args, **kwargs)
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/modeling_utils.py", line 4451, in from_pretrained
    model = cls(config, *model_args, **model_kwargs)
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/models/qwen3_omni_moe/modeling_qwen3_omni_moe.py", line 3802, in __init__
    super().__init__(config)
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/modeling_utils.py", line 1826, in __init__
    self.config._attn_implementation_internal = self._check_and_adjust_attn_implementation(
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/modeling_utils.py", line 2413, in _check_and_adjust_attn_implementation
    lazy_import_flash_attention(applicable_attn_implementation, force_import=True)
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/modeling_flash_attention_utils.py", line 136, in lazy_import_flash_attention
    _flash_fn, _flash_varlen_fn, _pad_fn, _unpad_fn = _lazy_imports(implementation)
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/modeling_flash_attention_utils.py", line 83, in _lazy_imports
    from flash_attn import flash_attn_func, flash_attn_varlen_func
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/flash_attn/__init__.py", line 3, in <module>
    from flash_attn.flash_attn_interface import (
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/flash_attn/flash_attn_interface.py", line 15, in <module>
    import flash_attn_2_cuda as flash_attn_gpu
ImportError: /root/miniconda3/envs/qwen/lib/python3.10/site-packages/flash_attn_2_cuda.cpython-310-x86_64-linux-gnu.so: undefined symbol: _ZNK3c106SymInt6sym_neERKS0_
```
调查了一下，原因是因为pytorch版本和我的flash-attention不匹配。flash-attention也得回退老版本。
回退老版本：
`pip install torch==2.7.0 torchvision==0.22.0 torchaudio==2.7.0 --index-url https://download.pytorch.org/whl/cu128`
不报上面的错了，但又有新错误了
```
(qwen) root@autodl-container-0a4d4f962f-c0ee9df0:~/autodl-tmp# python omni.py
Unrecognized keys in `rope_parameters` for 'rope_type'='default': {'mrope_section', 'mrope_interleaved', 'interleaved'}
Unrecognized keys in `rope_parameters` for 'rope_type'='default': {'mrope_section', 'interleaved'}
You are attempting to use Flash Attention 2 without specifying a torch dtype. This might lead to unexpected behaviour
Traceback (most recent call last):
  File "/root/autodl-tmp/omni.py", line 9, in <module>
    model = Qwen3OmniMoeForConditionalGeneration.from_pretrained(
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/modelscope/utils/hf_util/patcher.py", line 285, in from_pretrained
    module_obj = module_class.from_pretrained(
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/modeling_utils.py", line 270, in _wrapper
    return func(*args, **kwargs)
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/modeling_utils.py", line 4451, in from_pretrained
    model = cls(config, *model_args, **model_kwargs)
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/models/qwen3_omni_moe/modeling_qwen3_omni_moe.py", line 3807, in __init__
    self.enable_talker()
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/models/qwen3_omni_moe/modeling_qwen3_omni_moe.py", line 3812, in enable_talker
    self.code2wav = Qwen3OmniMoeCode2Wav._from_config(self.config.code2wav_config)
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/modeling_utils.py", line 270, in _wrapper
    return func(*args, **kwargs)
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/modeling_utils.py", line 2049, in _from_config
    model = cls(config, **kwargs)
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/models/qwen3_omni_moe/modeling_qwen3_omni_moe.py", line 3740, in __init__
    self.pre_transformer = Qwen3OmniMoeCode2WavTransformerModel._from_config(config)
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/modeling_utils.py", line 270, in _wrapper
    return func(*args, **kwargs)
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/modeling_utils.py", line 2049, in _from_config
    model = cls(config, **kwargs)
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/models/qwen3_omni_moe/modeling_qwen3_omni_moe.py", line 3569, in __init__
    self.rotary_emb = Qwen3OmniMoeRotaryEmbedding(config=config)
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/models/qwen3_omni_moe/modeling_qwen3_omni_moe.py", line 2435, in __init__
    self.rope_type = self.config.rope_parameters["rope_type"]
  File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/transformers/configuration_utils.py", line 198, in __getattribute__
    return super().__getattribute__(key)
AttributeError: 'Qwen3OmniMoeCode2WavConfig' object has no attribute 'rope_parameters'
```
新错误原因是因为当前transformers里面不支持rope_parameters。这个问题我觉得蛮奇怪的，因为我按照官方说法已经升级版本到最新的5.0.0dev版了，直接从源码安装的。实在不清楚为啥，因为transformers已经是最新的了。
我在github社区里找到了有人跟我一样的错误，根据他们反馈，修复了，但是有人说仍然没修复。[这个讨论](https://github.com/QwenLM/Qwen3-Omni/issues/93#issuecomment-3426749017)是围绕这个问题展开的。根据社区反馈，仍然有人遇到问题，没法跑。

改变思路，先不跑transformers，因为暂时问题不清楚。还是用VLLM部署方式
根据官方操作按装VLLM，
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
结果报错：
```
Traceback (most recent call last):
  File "/root/autodl-tmp/vllmTest.py", line 4, in <module>
    from vllm import LLM, SamplingParams
ImportError: cannot import name 'LLM' from 'vllm' (unknown location)
```
问题原因是把VLLm项目和测试文件放在同一个文件夹下，导致python索引扰乱了。解决方案就是先`pip uninstall vllm `然后在其他地方重新下载安装，这样问题就可以解决了。

解决完毕后继续执行测试文件，又发生报错：
```
INFO 11-03 15:47:45 [model_runner.py:1671] Graph capturing finished in 2 secs, took 0.11 GiB
INFO 11-03 15:47:45 [llm_engine.py:428] init engine (profile, create kv cache, warmup model) took 28.44 seconds
[rank0]: Traceback (most recent call last):
[rank0]:   File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/audioread/ffdec.py", line 142, in __init__
[rank0]:     self.proc = popen_multiple(
[rank0]:   File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/audioread/ffdec.py", line 92, in popen_multiple
[rank0]:     return subprocess.Popen(cmd, *args, **kwargs)
[rank0]:   File "/root/miniconda3/envs/qwen/lib/python3.10/subprocess.py", line 971, in __init__
[rank0]:     self._execute_child(args, executable, preexec_fn, close_fds,
[rank0]:   File "/root/miniconda3/envs/qwen/lib/python3.10/subprocess.py", line 1863, in _execute_child
[rank0]:     raise child_exception_type(errno_num, err_msg, err_filename)
[rank0]: FileNotFoundError: [Errno 2] No such file or directory: 'avconv'

[rank0]: During handling of the above exception, another exception occurred:

[rank0]: Traceback (most recent call last):
[rank0]:   File "/root/autodl-tmp/testvmml.py", line 47, in <module>
[rank0]:     audios, images, videos = process_mm_info(messages, use_audio_in_video=True)
[rank0]:   File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/qwen_omni_utils/v2_5/__init__.py", line 12, in process_mm_info
[rank0]:     audios = process_audio_info(conversations, use_audio_in_video)
[rank0]:   File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/qwen_omni_utils/v2_5/audio_process.py", line 75, in process_audio_info
[rank0]:     data = audioread.ffdec.FFmpegAudioFile(path)
[rank0]:   File "/root/miniconda3/envs/qwen/lib/python3.10/site-packages/audioread/ffdec.py", line 152, in __init__
[rank0]:     raise NotInstalledError()
[rank0]: audioread.ffdec.NotInstalledError
[rank0]:[W1103 15:47:55.564745187 ProcessGroupNCCL.cpp:1476] Warning: WARNING: destroy_process_group() was not called before program exit, which can leak resources. For more info, please see https://pytorch.org/docs/stable/distributed.html#shutdown (function operator())
```
这个报错的意思是我服务器里缺少FFmpeg音频处理库，所以我执行
`sudo apt install -y ffmpeg`

然后运行终于跑通了！

```
(qwen) root@autodl-container-0a4d4f962f-c0ee9df0:~/autodl-tmp# python testvmml.py 
INFO 11-03 15:50:29 [__init__.py:244] Automatically detected platform cuda.
Unrecognized keys in `rope_parameters` for 'rope_type'='default': {'interleaved', 'mrope_interleaved', 'mrope_section'}
Unrecognized keys in `rope_parameters` for 'rope_type'='default': {'interleaved', 'mrope_section'}
INFO 11-03 15:50:36 [config.py:841] This model supports multiple tasks: {'classify', 'reward', 'embed', 'generate'}. Defaulting to 'generate'.
`torch_dtype` is deprecated! Use `dtype` instead!
INFO 11-03 15:50:36 [config.py:1472] Using max model len 32768
INFO 11-03 15:50:37 [llm_engine.py:230] Initializing a V0 LLM engine (v0.9.3.dev6+gca66cbff0) with config: model='qwen3-omni-30b-int8', speculative_config=None, tokenizer='qwen3-omni-30b-int8', skip_tokenizer_init=False, tokenizer_mode=auto, revision=None, override_neuron_config={}, tokenizer_revision=None, trust_remote_code=True, dtype=torch.bfloat16, max_seq_len=32768, download_dir=None, load_format=auto, tensor_parallel_size=1, pipeline_parallel_size=1, disable_custom_all_reduce=False, quantization=compressed-tensors, enforce_eager=False, kv_cache_dtype=auto,  device_config=cuda, decoding_config=DecodingConfig(backend='auto', disable_fallback=False, disable_any_whitespace=False, disable_additional_properties=False, reasoning_backend=''), observability_config=ObservabilityConfig(show_hidden_metrics_for_version=None, otlp_traces_endpoint=None, collect_detailed_traces=None), seed=1234, served_model_name=qwen3-omni-30b-int8, num_scheduler_steps=1, multi_step_stream_outputs=True, enable_prefix_caching=None, chunked_prefill_enabled=False, use_async_output_proc=True, pooler_config=None, compilation_config={"level":0,"debug_dump_path":"","cache_dir":"","backend":"","custom_ops":[],"splitting_ops":[],"use_inductor":true,"compile_sizes":[],"inductor_compile_config":{"enable_auto_functionalized_v2":false},"inductor_passes":{},"use_cudagraph":false,"cudagraph_num_of_warmups":0,"cudagraph_capture_sizes":[8,4,2,1],"cudagraph_copy_inputs":false,"full_cuda_graph":false,"max_capture_size":8,"local_cache_dir":null}, use_cached_outputs=False, 
INFO 11-03 15:50:37 [cuda.py:363] Using Flash Attention backend.
INFO 11-03 15:50:37 [parallel_state.py:1076] rank 0 in world size 1 is assigned as DP rank 0, PP rank 0, TP rank 0, EP rank 0
INFO 11-03 15:50:37 [model_runner.py:1171] Starting to load model qwen3-omni-30b-int8...
You are attempting to use Flash Attention 2 without specifying a torch dtype. This might lead to unexpected behaviour
INFO 11-03 15:50:38 [compressed_tensors_wNa16.py:95] Using MarlinLinearKernel for CompressedTensorsWNA16
WARNING 11-03 15:50:38 [config.py:440] MoE DP setup unable to determine quantization scheme or unsupported quantization type. This model will not run with DP enabled.
INFO 11-03 15:50:38 [compressed_tensors_moe.py:82] Using CompressedTensorsWNA16MarlinMoEMethod
Loading safetensors checkpoint shards:   0% Completed | 0/9 [00:00<?, ?it/s]
Loading safetensors checkpoint shards:  11% Completed | 1/9 [00:00<00:03,  2.10it/s]
Loading safetensors checkpoint shards:  22% Completed | 2/9 [00:03<00:15,  2.26s/it]
Loading safetensors checkpoint shards:  33% Completed | 3/9 [00:07<00:17,  2.93s/it]
Loading safetensors checkpoint shards:  44% Completed | 4/9 [00:11<00:16,  3.25s/it]
Loading safetensors checkpoint shards:  56% Completed | 5/9 [00:15<00:13,  3.41s/it]
Loading safetensors checkpoint shards:  67% Completed | 6/9 [00:18<00:10,  3.42s/it]
Loading safetensors checkpoint shards:  78% Completed | 7/9 [00:22<00:07,  3.51s/it]
Loading safetensors checkpoint shards:  89% Completed | 8/9 [00:22<00:02,  2.43s/it]
Loading safetensors checkpoint shards: 100% Completed | 9/9 [00:24<00:00,  2.31s/it]
Loading safetensors checkpoint shards: 100% Completed | 9/9 [00:24<00:00,  2.72s/it]

INFO 11-03 15:51:03 [default_loader.py:272] Loading weights took 24.58 seconds
INFO 11-03 15:51:04 [model_runner.py:1203] Model loading took 33.0590 GiB and 26.101635 seconds
The image processor of type `Qwen2VLImageProcessor` is now loaded as a fast processor by default, even if the model checkpoint was saved with a slow processor. This is a breaking change and may produce slightly different outputs. To continue using the slow processor, instantiate this class with `use_fast=False`. Note that this behavior will be extended to all models in a future release.
WARNING 11-03 15:51:07 [model_runner.py:1368] Computed max_num_seqs (min(8, 32768 // 75264)) to be less than 1. Setting it to the minimum value of 1.
WARNING 11-03 15:51:10 [profiling.py:237] The sequence length used for profiling (max_num_batched_tokens / max_num_seqs = 32768) is too short to hold the multi-modal embeddings in the worst case (40989 tokens in total, out of which {'audio': 1170, 'image': 37632, 'video': 2187} are reserved for multi-modal embeddings). This may cause certain multi-modal inputs to fail during inference, even when the input text is short. To avoid this, you should increase `max_model_len`, reduce `max_num_seqs`, and/or reduce `mm_counts`.
/root/vllm/vllm/model_executor/layers/rotary_embedding.py:1658: UserWarning: To copy construct from a tensor, it is recommended to use sourceTensor.detach().clone() or sourceTensor.detach().clone().requires_grad_(True), rather than torch.tensor(sourceTensor).
  torch.tensor([1] * torch.tensor(video_grid_thw).shape[0]))
WARNING 11-03 15:51:20 [fused_moe.py:690] Using default MoE config. Performance might be sub-optimal! Config file not found at /root/vllm/vllm/model_executor/layers/fused_moe/configs/E=128,N=8192,device_name=NVIDIA_GeForce_RTX_4090.json
INFO 11-03 15:51:20 [marlin_utils.py:346] You are running Marlin kernel with bf16 on GPUs before SM90. You can consider change to fp16 to achieve better performance if possible.
INFO 11-03 15:51:26 [worker.py:294] Memory profiling takes 22.16 seconds
INFO 11-03 15:51:26 [worker.py:294] the current vLLM instance can use total_gpu_memory (47.37GiB) x gpu_memory_utilization (0.95) = 45.00GiB
INFO 11-03 15:51:26 [worker.py:294] model weights take 33.06GiB; non_torch_memory takes 0.08GiB; PyTorch activation peak memory takes 6.49GiB; the rest of the memory reserved for KV Cache is 5.38GiB.
INFO 11-03 15:51:27 [executor_base.py:113] # cuda blocks: 3671, # CPU blocks: 2730
INFO 11-03 15:51:27 [executor_base.py:118] Maximum concurrency for 32768 tokens per request: 1.79x
INFO 11-03 15:51:29 [model_runner.py:1513] Capturing cudagraphs for decoding. This may lead to unexpected consequences if the model is not static. To run the model in eager mode, set 'enforce_eager=True' or use '--enforce-eager' in the CLI. If out-of-memory error occurs during cudagraph capture, consider decreasing `gpu_memory_utilization` or switching to eager mode. You can also reduce the `max_num_seqs` as needed to decrease memory usage.
Capturing CUDA graph shapes: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4/4 [00:01<00:00,  2.07it/s]
INFO 11-03 15:51:31 [model_runner.py:1671] Graph capturing finished in 2 secs, took 0.11 GiB
INFO 11-03 15:51:31 [llm_engine.py:428] init engine (profile, create kv cache, warmup model) took 26.60 seconds
/root/miniconda3/envs/qwen/lib/python3.10/site-packages/librosa/core/audio.py:172: FutureWarning: librosa.core.audio.__audioread_load
        Deprecated as of librosa version 0.10.0.
        It will be removed in librosa version 1.0.
  y, sr_native = __audioread_load(path, offset, duration, dtype)
qwen-vl-utils using torchvision to read video.
/root/miniconda3/envs/qwen/lib/python3.10/site-packages/torchvision/io/_video_deprecation_warning.py:5: UserWarning: The video decoding and encoding capabilities of torchvision are deprecated from version 0.22 and will be removed in version 0.24. We recommend that you migrate to TorchCodec, where we'll consolidate the future decoding/encoding capabilities of PyTorch: https://github.com/pytorch/torchcodec
  warnings.warn(

Adding requests:   0%|                                                                                                                                                                                                                                           | 0/1 [00:00<?, ?it/s]
Adding requests: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1/1 [00:03<00:00,  3.84s/it]
Processed prompts:   0%|                                                                                                                                                                                     | 0/1 [00:00<?, ?it/s, est. speed input: 0.00 toks/s, output: 0.00 toks/s]/root/vllm/vllm/model_executor/layers/rotary_embedding.py:1658: UserWarning: To copy construct from a tensor, it is recommended to use sourceTensor.detach().clone() or sourceTensor.detach().clone().requires_grad_(True), rather than torch.tensor(sourceTensor).
  torch.tensor([1] * torch.tensor(video_grid_thw).shape[0]))
Processed prompts: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1/1 [00:01<00:00,  1.30s/it, est. speed input: 2732.53 toks/s, output: 66.91 toks/s]
Of course! Based on the image you provided, here is a detailed description of what is happening:

A person is using a stylus to draw a cartoon-style acoustic guitar on a tablet. The guitar is depicted with a light brown body, a black neck and headstock, and black strings. The person's left hand is holding the tablet steady on a wooden table, while their right hand actively uses the stylus to draw.
```
# HOW TO HOST LLM LOCALLY?

## 1. Using vLLM

### Conceptual Flow

```
Windows Host
 └── WSL2 Linux Kernel
      └── Docker Engine
           └── vLLM Inference Container
                └── CUDA Runtime
                     └── PyTorch
                          └── Qwen Model
                               └── OpenAI API Server
```

### Steps to locally run and host LLM on your laptop

- Use or install WSL2 if running on windows
- Upgrade or install drivers from the official website (>= CUDA 13 & 55x. GTX 1650)
- Download a small model in .safetensor format (NOT GGUF) (example: Qwen2.5-0.5Instruct) ideally using huggingface-cli (Manual download doesn't work properly)

```
huggingface-cli download Qwen/Qwen2.5-0.5B-Instruct --local-dir C:\models\Qwen2.5-0.5B-Instruct
```

- Use the stored model to spin up docker container

```
docker run --rm -it --gpus all -p 8000:8000 -v C:\models:/models vllm/vllm-openai:latest --model /models/Qwen2.5-0.5B-Instruct --dtype float16 --max-model-len 1024 --gpu-memory-utilization 0.75 --enforce-eager --swap-space 1
```

It is important to set parameters such as max-model-len, gpu-memory-utilization, swap-space to avoid out of memory errors

```
“No available memory for the cache blocks.” Available KV cache memory: -0.88 GiB
```

- Use postman or custom script to invoke the endpoint from : http://localhost:8000/v1/chat/completions

#### List of errors you might encounter

1. `FA2 is only supported on devices with compute capability >= 8`

**Fix: Use `--enforce-eager` flag to fall back from FlashAttention2 to eager kernels**

2. `4.00 GiB out of the 2.81 GiB total CPU memory is allocated for the swap space`

**Fix: Use `--swap-space 1` flag to limit the CPU allocated by vLLM for KV cache spillover, scheduler buffer**

3. `No available memory for the cache blocks: Available KV cache memory: -0.88 GiB`

**Fix: Use `--max-model-len 1024` and `--gpu-memory-utilization 0.75` to limit the VRAM used by the model**

4. `cuda>=12.9 required`

**Fix: Update the CUDA driver**

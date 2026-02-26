# vLLM info and tips

## Running

- Start vLLM server (EP1, gpt2)
  ```
  python3.12 -m vllm.entrypoints.openai.api_server --model openai-community/gpt2 --host 127.0.0.1 --port 8000
  ```
- Start vLLM server (EP2, gpt2)
  ```
  vllm serve openai-community/gpt2 --host 127.0.0.1 --port 8000
  ```
- Start vLLM server (EP2, Qwen2.5)
  ```
  vllm serve Qwen/Qwen2.5-0.5B-Instruct --host 127.0.0.1 --dtype float32
  ```
- Run completion query on gpt2
  ```
  curl http://localhost:8000/v1/completions -H "Content-Type: application/json" \
      -d '{"model": "openai-community/gpt2", "prompt": "The capital of Israel is the city of", "max_tokens": 2, "temperature": 0.0}'
  ```
- Run chat prompt on Qwen2.5 (1)
  ```
  curl http://127.0.0.1:8000/v1/chat/completions -H "Content-Type: application/json" \
      -d '{"model": "Qwen/Qwen2.5-0.5B-Instruct", "messages": [{"role": "user", "content": "What is the capital of France?"}], "max_tokens": 50, "temperature": 0 }'
  ```
- Run chat prompt on Qwen2.5 (2)
  ```
  curl http://127.0.0.1:8000/v1/chat/completions -H "Content-Type: application/json" \
      -d '{ "model": "Qwen/Qwen2.5-0.5B-Instruct", "messages": [{"role": "user", "content": "I live next to the carwash. For washing my car, should I walk or drive to the carwash? Be concise."}], "max_tokens": 200, "temperature": 0 }'
  ```

## Building

- Build and install local vLLM to system editable package for CPU:
  ```
  pip install torch torchvision --index-url https://download.pytorch.org/whl/cpu --force-reinstall
  pip install --upgrade setuptools
  MAX_JOBS=4 VLLM_TARGET_DEVICE=cpu python3 -m pip install -e . --no-build-isolation
  ```
  Notes:
  1. Using no build isolation (local env) to pick right torch
  2. Must install CPU-only PyTorch first — CUDA-built PyTorch's cmake config
     requires CUDA libraries even for CPU-only vLLM builds
  3. Modify `requirements/cpu*` to require `torch>=2.8.0`
  4. `MAX_JOBS` caps the number of parallel CMake build jobs (useful to limit memory usage)
  5. Requires avx512. Otherwise, apply this patch:
     https://github.com/vllm-project/vllm/issues/33991

- Full vLLM build instructions for CPU: https://docs.vllm.ai/en/stable/getting_started/installation/cpu

## Chatting

`chat.py` - sample script for continuous chat:

```python
from openai import OpenAI
client = OpenAI(base_url="http://127.0.0.1:8000/v1", api_key="unused")
messages = []
while True:
    user_input = input("You: ")
    if user_input.lower() in ("quit", "exit"):
        break
    messages.append({"role": "user", "content": user_input})
    response = client.chat.completions.create(
        model="Qwen/Qwen2.5-0.5B-Instruct",
        messages=messages,
        temperature=0.7
    )
    reply = response.choices[0].message.content
    print(f"Bot: {reply}")
    messages.append({"role": "assistant", "content": reply})
```

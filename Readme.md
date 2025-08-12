# How to run llama.cpp with Docker or Kubernetes
When comparing vLLM and Ollama, the main appeal of llama.cpp lies in its exceptional portability and wide-ranging hardware support.

llama.cpp is specifically engineered to run on the CPU alone, without needing a dedicated GPU or special drivers. This gives it a high degree of compatibility, allowing it to run not only on major operating systems like macOS, Windows, and Linux but also on smaller devices such as the Raspberry Pi. While vLLM is primarily dependent on NVIDIA GPUs, llama.cpp is optimized to run efficiently on Apple Silicon (M1/M2/M3/M4), which is my case, and standard CPUs, making it accessible to a much wider audience.

# LLM Runtime Comparison: llama.cpp vs Ollama vs vLLM

| | **llama.cpp** | **Ollama** | **vLLM** |
| :--- | :--- | :--- | :--- |
| **Primary Use Case** | Local inference on a wide range of hardware (especially CPU-centric) | Simplified local deployment and sharing of models | High-throughput, low-latency serving of LLMs (GPU-centric) |
| **Hardware Support** | **Excellent**. Optimized for CPU, Apple Silicon (M1/M2/M3), and can utilize GPUs. | **Good**. Primarily supports CPU/GPU (NVIDIA, AMD, Apple Silicon). Simplifies the setup process. | **Excellent but limited**. Highly optimized for NVIDIA GPUs. Emerging support for AMD and other GPUs, but less mature. |
| **Model Format** | **GGUF** (GPT-Generated Unified Format) | **Custom format** for local serving. Pulls pre-packaged models from a registry. | Standard Hugging Face model formats (e.g., PyTorch, SafeTensors) |
| **Ease of Use** | **Requires compilation, but is straightforward**. Simple command-line execution. | **Easiest**. A single executable to download and run. Offers a Docker-like experience. | **Moderate**. Requires a Python environment setup, dependencies, and some configuration. |
| **Performance** | **Good on CPU, excellent on Apple Silicon**. Performance is highly dependent on hardware but is heavily optimized for efficiency. | **Good**. Offers solid performance for local use. | **Excellent**. Designed for maximum throughput and efficiency on GPUs, especially for multiple concurrent requests. |
| **Key Advantage** | **Portability and hardware flexibility**. Can run on virtually any modern computer, even without a dedicated GPU. | **Simplicity**. Abstracts away the complexities of running models locally, making it a great entry point for beginners. | **Speed and scale**. The fastest option for high-volume inference on powerful GPUs. Ideal for production environments. |

<br>

| | **llama.cpp** | **Ollama** | **vLLM** |
| :--- | :--- | :--- | :--- |
| **主な用途** | 幅広いハードウェアでのローカル推論（特にCPU中心） | モデルのローカル展開と共有の簡素化 | 高スループット・低遅延のLLMサービス提供（GPU中心） |
| **ハードウェアサポート** | **優れている**。CPU、Apple Silicon（M1/M2/M3）向けに最適化されており、GPUも利用可能。 | **良い**。主にCPU/GPU（NVIDIA, AMD, Apple Silicon）をサポート。セットアップが簡単。 | **優れているが限定的**。NVIDIA GPU向けに高度に最適化。他のGPUサポートも登場しているが、まだ成熟度は低い。 |
| **モデル形式** | **GGUF** (GPT-Generated Unified Format) | ローカルサーバー用の**独自のカスタム形式**。レジストリからプリパッケージされたモデルをダウンロードする。 | 標準的なHugging Faceモデル形式（例：PyTorch, SafeTensors） |
| **使いやすさ** | **コンパイルが必要だが、簡単**。シンプルなコマンドライン実行。 | **最も簡単**。ダウンロードして実行する単一の実行可能ファイル。Dockerのような体験。 | **中程度**。Python環境、依存関係、いくつかの設定が必要。 |
| **パフォーマンス** | **CPUでは良好、Apple Siliconでは非常に優れている**。ハードウェアに大きく依存するが、効率性のために高度に最適化されている。 | **良い**。ローカルでの使用には堅実なパフォーマンスを提供。 | **非常に優れている**。複数の同時リクエストに対して、GPUでの最大スループットと効率性を実現するように設計されている。 |
| **主な強み** | **移植性とハードウェアの柔軟性**。専用のGPUがなくても、事実上あらゆる最新のコンピュータで実行できる。 | **シンプルさ**。ローカルでモデルを実行する複雑さを抽象化し、初心者にとって優れた入口となる。 | **速度と拡張性**。強力なGPU上で大量の推論を行うための最速の選択肢。本番環境に最適。 |


---

### In-Depth Breakdown

#### llama.cpp
The primary appeal of **llama.cpp** is its **portability and hardware flexibility**. It was designed to run on a wide variety of devices without the complex setup of Python or GPUs. A key strength is its efficiency on Apple Silicon and general-purpose CPUs. Its custom model format, GGUF, simplifies quantization (a technique to reduce model size) and improves memory efficiency.

#### Ollama
**Ollama** builds on the technology of llama.cpp to create a significantly

# 1. Run gguf model with Docker
# 1-1. Building Docker image
```
docker build -t llama-cpp-python:arm64 .
```

# 1-2. Download gguf model
```
cd
mkdir models
cd models
wget https://huggingface.co/bartowski/Llama-3.2-1B-Instruct-GGUF/resolve/main/Llama-3.2-1B-Instruct-Q4_K_M.gguf
```

# 1-3. Run the docker image with gguf
```
docker run -d --rm --name=test -p 8000:8000 -v "$(pwd)/models:/app/models" llama-cpp-python:arm64 python3 -m llama_cpp.server --host 0.0.0.0 --port 8000 --model /app/models/Llama-3.2-1B-Instruct-Q4_K_M.gguf
```

# 1-4. Confirm the response from the llama_cpp server
```
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "user",
        "content": "Who is Tokugawa Yoshinobu ?"
      }
    ],
    "stream": false,
    "max_tokens": 128
  }'
```
# 2. Run gguf model with Kubernetes

# 2-1. Create PVC and PV
```
kubectl apply -f pvc-nfs-openwebui.yaml 
kubectl apply -f pvc-nfs-llama-cpp-python.yaml 
```

# 2-2. Create Pod for llama.cpp python
```
 kubectl apply -f llama-cpp-python.yaml
```

This yaml will start to download the gguf model before creating Pod for llama.cpp python.
```
vagrant@nfs:~/exports/pvc-6e2b830f-2863-4fca-b2d5-178d0d9e55a2$ ls -trl
total 788772
-rw-r--r-- 1 nobody nogroup 807694464 Aug 12 14:58 Llama-3.2-1B-Instruct-Q4_K_M.gguf
```

# 2-3. Create Pod for Open WebUI
```
kubectl apply -f openwebui-llama-cpp-python.yaml
```

# 2-4. Confirm the External IP exposed outside of Cluster
```
kubectl get services
```
```
vagrant@master:~/openwebui-llamacpp$ kubectl get services
NAME                              TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)          AGE
kubernetes                        ClusterIP      10.96.0.1       <none>         443/TCP          3h39m
svc-llama-cpp-python              ClusterIP      10.99.202.17    <none>         8000/TCP         3m59s
svc-open-webui-llama-cpp-python   LoadBalancer   10.110.39.204   192.168.33.2   8080:31658/TCP   44s
```

# 2-5. Access to the External IP with some Browsers
<img src="https://github.com/developer-onizuka/openwebui-llamacpp/blob/main/access-openwebui.png" width="720">

You can use these kinds of gguf model so easily through llama.cpp python framework.<br>

<img src="https://github.com/developer-onizuka/openwebui-llamacpp/blob/main/Llama-3.2-1B-Instruct-Q4_K_M.png" width="720">

# How to run llama.cpp with Docker or Kubernetes

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

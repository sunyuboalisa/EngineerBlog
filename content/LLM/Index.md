---
longform:
  format: scenes
  title: LLM
  workflow: Default Workflow
  sceneFolder: /
  scenes:
    - python
  ignoredFiles: []
---
```
uv venv --python 3.12 --seed
source .venv/bin/activate
pip install vllm

```

```
export HF_ENDPOINT="https://hf-mirror.com/" 
export HF_HUB_DOWNLOAD_TIMEOUT=300
export HF_HUB_ETAG_TIMEOUT=1800

uv run hf download "Qwen/Qwen3-8B" --local-dir ./qwen3-folder --max-workers 1
```


```
# 测试是否能运行成功
uv run vllm serve ./qwen3-4b-awq-folder/ \
    --gpu-memory-utilization 0.75 \
    --max-model-len 8192 \
    --max-num-seqs 16 \
    --served-model-name qwen3
```

```
# 配置端口映射
hostname -I

netsh interface portproxy add v4tov4 listenport=2224 listenaddress=0.0.0.0 connectport=22 connectaddress=172.23.161.207 protocol=tcp

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=8002 connectaddress=172.23.161.207 connectport=8002

netsh interface portproxy show all
```

```
# 创建启动脚本
nano start_vllm.sh

#!/bin/bash # 进入你的项目目录（包含 qwen3 文件夹的目录） 
cd ~ # 使用 uv 运行命令 
# nohup ... & 确保它在后台运行，并将日志记入 vllm.log 
nohup uv run vllm serve ./qwen3-4b-awq-folder/ \
  --gpu-memory-utilization 0.75 \
  --max-model-len 8192 \
  --max-num-seqs 16 \
  --served-model-name qwen3 > ~/vllm.log 2>&1 &

# 然后赋予权限  
chmod +x ~/start_vllm.sh

# 配置 wsl 中自启动
sudo nano /etc/wsl.conf
[boot]
systemd=true
command="su - alisa -c /home/alisa/start_vllm.sh"
```




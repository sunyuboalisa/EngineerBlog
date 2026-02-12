- pipx
- uv

uv 配置地址
windows 
%APPDATA%\uv\uv.toml
linux 
用户目录下/.config/uv/uv.toml

配置如下
```
[[index]]
url = "https://mirrors.aliyun.com/pypi/"

default = true
```

```
sudo apt update 
sudo apt install pipx 
pipx ensurepath

pipx install uv
pipx ensurepath
source ~/.bashrc

echo 'eval "$(uv generate-shell-completion bash)"' >> ~/.bashrc
echo 'eval "$(uvx --generate-shell-completion bash)"' >> ~/.bashrc
```
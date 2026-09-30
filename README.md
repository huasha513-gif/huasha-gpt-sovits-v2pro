# 华莎 GPT-SoVITS v2Pro 声音模型

夜凛的声音克隆模型，基于 GPT-SoVITS v2Pro 训练。

## 📥 模型下载

模型文件托管在 HuggingFace：

👉 https://huggingface.co/huasha513/gpt-sovits-v2pro

## 🛠️ 使用方法

1. 克隆 GPT-SoVITS 项目：
git clone https://github.com/RVC-Boss/GPT-SoVITS

2. 下载模型文件放入对应目录：
- G_30888.pth → SoVITS权重
- s1_531.ckpt → GPT权重
- ref.wav → 参考音频

3. 启动推理服务：
python api_v2.py -p 9880

## 📋 环境要求
- Python 3.10+
- CUDA GPU 推荐
- pip install -r requirements.txt

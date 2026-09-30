# GPT-SoVITS v2Pro 声音克隆权重包

基于 [GPT-SoVITS](https://github.com/RVC-Boss/GPT-SoVITS) v2Pro 版本训练的中文声音克隆模型权重。133 条音频训练，可用于中文语音合成（TTS）。

## 模型文件

托管在 HuggingFace: [huasha513/gpt-sovits-v2pro](https://huggingface.co/huasha513/gpt-sovits-v2pro)

| 文件 | 大小 | 说明 |
|------|------|------|
| `G_10880.pth` | ~907 MB | SoVITS (S2) 权重，训练 160 步 |
| `s1_133_v2pro_20260320-e20.ckpt` | ~148 MB | GPT (S1) 权重，133 条音频，epoch 20 |
| `ref.wav` | ~530 KB | 推理参考音频（约 5 秒，普通话） |

## 快速开始

### 1. 安装 GPT-SoVITS v2Pro

必须使用 `20250606v2pro` 标签版本，其他版本的 S1 权重格式不兼容。

```bash
git clone https://github.com/RVC-Boss/GPT-SoVITS
cd GPT-SoVITS
git checkout 20250606v2pro
pip install -r requirements.txt
```

**环境要求：**
- Python 3.10+
- NVIDIA GPU + CUDA（推荐；CPU 也能跑但很慢）
- PyTorch 2.x（安装 GPT-SoVITS 依赖时会自动装）

### 2. 下载模型文件

从 HuggingFace 下载：

```bash
# 方式一：用 huggingface-cli
pip install huggingface_hub
huggingface-cli download huasha513/gpt-sovits-v2pro --local-dir ./my_weights

# 方式二：用 git lfs
git lfs install
git clone https://huggingface.co/huasha513/gpt-sovits-v2pro ./my_weights
```

### 3. 文件放置

下载后目录结构如下，三个文件放在同一个目录即可：

```
my_weights/
├── G_10880.pth                        # SoVITS 权重
├── s1_133_v2pro_20260320-e20.ckpt     # GPT 权重
└── ref.wav                            # 参考音频
```

配置文件 `s2v2Pro.json` 无需单独下载，GPT-SoVITS 代码仓自带：
```
GPT-SoVITS/GPT_SoVITS/configs/s2v2Pro.json
```

### 4. 启动推理服务

```bash
cd GPT-SoVITS

# 方式一：启动时指定权重路径
python api_v2.py -p 9886 \
  -s /path/to/my_weights/G_10880.pth \
  -g /path/to/my_weights/s1_133_v2pro_20260320-e20.ckpt

# 方式二：先启动，再通过 API 加载权重
python api_v2.py -p 9886
```

服务启动后监听 `http://127.0.0.1:9886`。

如果使用方式二，通过 API 加载权重：

```bash
# 加载 GPT (S1) 权重
curl "http://127.0.0.1:9886/set_gpt_weights?weights_path=/path/to/my_weights/s1_133_v2pro_20260320-e20.ckpt"

# 加载 SoVITS (S2) 权重
curl "http://127.0.0.1:9886/set_sovits_weights?weights_path=/path/to/my_weights/G_10880.pth"
```

### 5. 调用推理

**参考文本**（必须与 `ref.wav` 完全一致）：
```
老公比谁都明白。你不是因为空虚才靠近我，不是因为欲望才说你爱我，不是因为缺什么才抓我做补丁。
```

#### curl 示例

```bash
curl -X POST http://127.0.0.1:9886/tts \
  -H "Content-Type: application/json" \
  -d '{
    "text": "你好，这是一段测试语音。",
    "text_lang": "zh",
    "ref_audio_path": "/path/to/my_weights/ref.wav",
    "prompt_text": "老公比谁都明白。你不是因为空虚才靠近我，不是因为欲望才说你爱我，不是因为缺什么才抓我做补丁。",
    "prompt_lang": "zh",
    "top_k": 20,
    "top_p": 0.6,
    "temperature": 0.6,
    "text_split_method": "cut0",
    "batch_size": 1,
    "batch_threshold": 0.75,
    "split_bucket": true,
    "speed_factor": 1.0,
    "fragment_interval": 0.3,
    "seed": -1,
    "media_type": "wav",
    "streaming_mode": false,
    "parallel_infer": true,
    "repetition_penalty": 1.35,
    "sample_steps": 8,
    "super_sampling": false
  }' --output output.wav
```

#### Python 示例

```python
import requests

payload = {
    "text": "你好，这是一段测试语音。",
    "text_lang": "zh",
    "ref_audio_path": "/path/to/my_weights/ref.wav",
    "aux_ref_audio_paths": [],
    "prompt_text": "老公比谁都明白。你不是因为空虚才靠近我，不是因为欲望才说你爱我，不是因为缺什么才抓我做补丁。",
    "prompt_lang": "zh",
    "top_k": 20,
    "top_p": 0.6,
    "temperature": 0.6,
    "text_split_method": "cut0",
    "batch_size": 1,
    "batch_threshold": 0.75,
    "split_bucket": True,
    "speed_factor": 1.0,
    "fragment_interval": 0.3,
    "seed": -1,
    "media_type": "wav",
    "streaming_mode": False,
    "parallel_infer": True,
    "repetition_penalty": 1.35,
    "sample_steps": 8,
    "super_sampling": False,
}

resp = requests.post("http://127.0.0.1:9886/tts", json=payload)
with open("output.wav", "wb") as f:
    f.write(resp.content)
```

## 参数说明

| 参数 | 推荐值 | 说明 |
|------|--------|------|
| `speed_factor` | 1.0 | 语速，可调 0.6~1.5 |
| `sample_steps` | 8 | 采样步数，8~32，越高越慢越精细 |
| `temperature` | 0.6 | 采样温度，偏低更稳定 |
| `top_k` | 20 | 采样范围 |
| `top_p` | 0.6 | 核采样阈值 |
| `repetition_penalty` | 1.35 | 抑制重复发音 |
| `text_split_method` | cut0 | 按中文句号切分，适合长文本 |
| `fragment_interval` | 0.3 | 句间停顿（秒） |

最常调的两个参数是 `speed_factor`（语速）和 `sample_steps`（质量/速度权衡），其余保持默认即可。

## 参考音频说明

`ref.wav` 是推理时的声音参考，约 5 秒普通话。推理时 `prompt_text` 必须与 `ref.wav` 的内容完全一致，否则会影响合成质量。

如果你有自己的参考音频，可以替换 `ref.wav` 并同步更新 `prompt_text`，但需注意：
- 参考音频建议 3~10 秒
- 音质清晰、无背景噪音
- 参考文本与音频内容逐字对应

## 许可

⚠️ **本模型仅供个人学习和研究使用，不可商用。** 声音素材克隆自 AI 语音助手的语音输出，原始声音版权归 OpenAI 所有，本项目不拥有该声音的任何权利。

模型权重基于 GPT-SoVITS 项目训练，请同时遵循 [GPT-SoVITS 的许可协议](https://github.com/RVC-Boss/GPT-SoVITS/blob/main/LICENSE)。

---

# GPT-SoVITS v2Pro Voice Cloning Weights

Voice cloning model weights trained with [GPT-SoVITS](https://github.com/RVC-Boss/GPT-SoVITS) v2Pro. Trained on 133 audio samples for Chinese (Mandarin) text-to-speech synthesis.

## Model Files

Hosted on HuggingFace: [huasha513/gpt-sovits-v2pro](https://huggingface.co/huasha513/gpt-sovits-v2pro)

| File | Size | Description |
|------|------|-------------|
| `G_10880.pth` | ~907 MB | SoVITS (S2) weights, trained 160 steps |
| `s1_133_v2pro_20260320-e20.ckpt` | ~148 MB | GPT (S1) weights, 133 audio samples, epoch 20 |
| `ref.wav` | ~530 KB | Reference audio for inference (~5s, Mandarin Chinese) |

## Quick Start

### 1. Install GPT-SoVITS v2Pro

You **must** use the `20250606v2pro` tag. Other versions have incompatible S1 weight formats.

```bash
git clone https://github.com/RVC-Boss/GPT-SoVITS
cd GPT-SoVITS
git checkout 20250606v2pro
pip install -r requirements.txt
```

**Requirements:**
- Python 3.10+
- NVIDIA GPU + CUDA (recommended; CPU works but is slow)
- PyTorch 2.x (installed automatically with dependencies)

### 2. Download Model Files

```bash
# Option A: huggingface-cli
pip install huggingface_hub
huggingface-cli download huasha513/gpt-sovits-v2pro --local-dir ./my_weights

# Option B: git lfs
git lfs install
git clone https://huggingface.co/huasha513/gpt-sovits-v2pro ./my_weights
```

### 3. File Layout

All three files go in the same directory:

```
my_weights/
├── G_10880.pth                        # SoVITS weights
├── s1_133_v2pro_20260320-e20.ckpt     # GPT weights
└── ref.wav                            # Reference audio
```

The config file `s2v2Pro.json` ships with the GPT-SoVITS codebase:
```
GPT-SoVITS/GPT_SoVITS/configs/s2v2Pro.json
```

### 4. Start the Inference Server

```bash
cd GPT-SoVITS

# Option A: specify weights at startup
python api_v2.py -p 9886 \
  -s /path/to/my_weights/G_10880.pth \
  -g /path/to/my_weights/s1_133_v2pro_20260320-e20.ckpt

# Option B: start first, load weights via API
python api_v2.py -p 9886
```

The server listens on `http://127.0.0.1:9886`.

If using Option B, load weights via API:

```bash
# Load GPT (S1) weights
curl "http://127.0.0.1:9886/set_gpt_weights?weights_path=/path/to/my_weights/s1_133_v2pro_20260320-e20.ckpt"

# Load SoVITS (S2) weights
curl "http://127.0.0.1:9886/set_sovits_weights?weights_path=/path/to/my_weights/G_10880.pth"
```

### 5. Run Inference

**Reference text** (must match `ref.wav` exactly):
```
老公比谁都明白。你不是因为空虚才靠近我，不是因为欲望才说你爱我，不是因为缺什么才抓我做补丁。
```

#### curl Example

```bash
curl -X POST http://127.0.0.1:9886/tts \
  -H "Content-Type: application/json" \
  -d '{
    "text": "你好，这是一段测试语音。",
    "text_lang": "zh",
    "ref_audio_path": "/path/to/my_weights/ref.wav",
    "prompt_text": "老公比谁都明白。你不是因为空虚才靠近我，不是因为欲望才说你爱我，不是因为缺什么才抓我做补丁。",
    "prompt_lang": "zh",
    "top_k": 20,
    "top_p": 0.6,
    "temperature": 0.6,
    "text_split_method": "cut0",
    "batch_size": 1,
    "batch_threshold": 0.75,
    "split_bucket": true,
    "speed_factor": 1.0,
    "fragment_interval": 0.3,
    "seed": -1,
    "media_type": "wav",
    "streaming_mode": false,
    "parallel_infer": true,
    "repetition_penalty": 1.35,
    "sample_steps": 8,
    "super_sampling": false
  }' --output output.wav
```

#### Python Example

```python
import requests

payload = {
    "text": "你好，这是一段测试语音。",
    "text_lang": "zh",
    "ref_audio_path": "/path/to/my_weights/ref.wav",
    "aux_ref_audio_paths": [],
    "prompt_text": "老公比谁都明白。你不是因为空虚才靠近我，不是因为欲望才说你爱我，不是因为缺什么才抓我做补丁。",
    "prompt_lang": "zh",
    "top_k": 20,
    "top_p": 0.6,
    "temperature": 0.6,
    "text_split_method": "cut0",
    "batch_size": 1,
    "batch_threshold": 0.75,
    "split_bucket": True,
    "speed_factor": 1.0,
    "fragment_interval": 0.3,
    "seed": -1,
    "media_type": "wav",
    "streaming_mode": False,
    "parallel_infer": True,
    "repetition_penalty": 1.35,
    "sample_steps": 8,
    "super_sampling": False,
}

resp = requests.post("http://127.0.0.1:9886/tts", json=payload)
with open("output.wav", "wb") as f:
    f.write(resp.content)
```

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `speed_factor` | 1.0 | Speech speed, adjustable 0.6~1.5 |
| `sample_steps` | 8 | Sampling steps, 8~32; higher = slower but finer quality |
| `temperature` | 0.6 | Sampling temperature; lower = more stable |
| `top_k` | 20 | Top-k sampling range |
| `top_p` | 0.6 | Nucleus sampling threshold |
| `repetition_penalty` | 1.35 | Suppresses repeated phonemes |
| `text_split_method` | cut0 | Splits by Chinese period; good for long text |
| `fragment_interval` | 0.3 | Pause between sentences (seconds) |

The two most commonly tuned parameters are `speed_factor` (speech rate) and `sample_steps` (quality vs. speed tradeoff). The rest can stay at their defaults.

## Reference Audio Notes

`ref.wav` is the voice reference for inference, ~5 seconds of Mandarin speech. The `prompt_text` must match `ref.wav` word-for-word; mismatches degrade synthesis quality.

To use your own reference audio:
- Keep it 3~10 seconds long
- Clean audio, no background noise
- `prompt_text` must be an exact transcription

## License

⚠️ **This model is for personal learning and research only. Commercial use is prohibited.** The voice is cloned from an AI voice assistant's speech output. Copyright of the original voice belongs to OpenAI. This project does not claim any rights to the voice.

Weights trained using the GPT-SoVITS framework. Please also follow the [GPT-SoVITS license](https://github.com/RVC-Boss/GPT-SoVITS/blob/main/LICENSE).

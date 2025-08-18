---
layout: post
author: Dragroo
title: ai字幕生成教程（韩语->中文）
tag: 传家小技巧
---
1. [ 下载最新版PotPlayer ](https://t1.daumcdn.net/potplayer/PotPlayer/Version/Latest/PotPlayerSetup64.exe)

2. 选择`字幕-创建有声字幕-创建有声字幕...`，转换引擎选择Whisper-Faster，型号选择small，并点击对应的下载按钮（也可以使用[hugging-face](https://huggingface.co/Systran/faster-whisper-small)下载。下载完成后，点击开始，注意必须要先播放一个视频。

3. 关闭生成字幕窗口，选择`字幕-保存字幕-字幕另存为`保存字幕。

4. 使用deepseek、通义千问等API，进行翻译

```python
import os
from openai import OpenAI
from tqdm import tqdm

# 初始化DeepSeek API客户端
client = OpenAI(
    api_key="sk-61758a71456547bb9a920f280539f7cf",
    base_url="https://api.deepseek.com"
)

def translate_text(text):
    """使用DeepSeek API翻译文本"""
    if not text.strip():
        return text
        
    try:
        response = client.chat.completions.create(
            model="deepseek-chat",
            messages=[
                {
                    "role": "system",
                    "content": "你是一名专业字幕翻译员。请将以下内容准确翻译成简体中文，保持口语化表达，不要添加额外解释。"
                },
                {
                    "role": "user",
                    "content": f"翻译这段字幕内容，保持原格式不变:\n{text}"
                }
            ],
            temperature=0.3,
            stream=False
        )
        return response.choices[0].message.content.strip()
    except Exception as e:
        print(f"翻译出错: {e}")
        return text  # 出错时返回原文

def process_srt_file(input_file, output_file):
    """处理SRT文件并生成翻译版本"""
    with open(input_file, 'r', encoding='utf-8') as f:
        lines = f.readlines()
    
    block_count = 0
    for line in lines:
        if line.strip().isdigit():
            block_count += 1

    new_lines = []
    i = 0
    total = len(lines)

    pbar = tqdm (total=block_count, desc="翻译进度", unit="block")
    while i < total:
        # 保留序号行
        if lines[i].strip().isdigit():
            new_lines.append(lines[i])
            i += 1
            continue
        
        # 保留时间轴行 (包含 -->)
        if '-->' in lines[i]:
            new_lines.append(lines[i])
            i += 1
            continue
        
        # 处理字幕文本行
        text_block = []
        while i < total and lines[i].strip() != '':
            text_block.append(lines[i])
            i += 1
        
        # 翻译字幕文本
        if text_block:
            original_text = ''.join(text_block)
            translated_text = translate_text(original_text)
            new_lines.append(translated_text + '\n')
            pbar.update(1)
        
        # 保留空行
        if i < total and lines[i].strip() == '':
            new_lines.append('\n')
            i += 1
    
    pbar.close()
    # 写入新文件
    with open(output_file, 'w', encoding='utf-8') as f:
        f.writelines(new_lines)
    print(f"翻译完成! 已保存到: {output_file}")

if __name__ == "__main__":
    input_srt = "subtitle.srt"  # 输入文件名
    output_srt = "subtitle_zh.srt"  # 输出文件名
    
    if not os.path.exists(input_srt):
        print(f"错误: 文件 {input_srt} 不存在")
    else:
        print("开始翻译字幕...")
        process_srt_file(input_srt, output_srt)

```
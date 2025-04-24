# 微调Whisper语音识别模型和加速推理

简体中文 | [English](./README_en.md)


## 前言

本项目的主要目标是对 Whisper 模型应用 LoRA 进行微调。由于原作者已对相关内容进行了详尽的介绍，特此致谢并推荐参考原项目获取更多细节[Whisper-Finetune
](https://github.com/yeyupiaoling/Whisper-Finetune)。本文仅介绍训练过程、加速推理方法和对于SAP的数据筛选和评估，以便集成至 [WhisperX](https://github.com/m-bain/whisperX) 框架中。



<a name='准备数据'></a>

## 准备数据

需要准备一个jsonlines的数据列表，数据格式如下。对于SAP数据集的准备请参考`aishell.py`。


```json
{
   "audio": {
      "path": "dataset/0.wav"
   },
   "sentence": "近几年，不但我用书给女儿压岁，也劝说亲朋不要给女儿压岁钱，而改送压岁书。",
   "language": "Chinese",
   "duration": 7.37
}
```

<a name='微调模型'></a>

## 微调模型

<a name='单卡训练'></a>

单卡训练

```shell
CUDA_VISIBLE_DEVICES=0 python finetune.py
```

<a name='多卡训练'></a>

多卡训练：使用torchrun启动多卡训练，命令如下，通过`--nproc_per_node`指定使用的显卡数量。
```shell
torchrun --nproc_per_node=2 finetune.py
```

<a name='合并模型'></a>

## 合并模型

微调完成之后会有两个模型，第一个是Whisper基础模型，第二个是Lora模型。
```shell
python merge_lora.py --lora_model=output/whisper-tiny/checkpoint-best/ --output_dir=models/
```

合并后的模型可以用来预测：

```shell
python infer.py --audio_path=dataset/test.wav --model_path=models/whisper-tiny-finetune
```

## 加速推理

把合并后的模型转换为CTranslate2模型
```shell
ct2-transformers-converter --model models/whisper-tiny-finetune --output_dir models/whisper-tiny-finetune-ct2 --copy_files tokenizer.json preprocessor_config.json --quantization float16
```

使用Ctranslate2格式模型预测
```shell
python infer_ct2.py --audio_path=dataset/test.wav --model_path=models/whisper-tiny-finetune-ct2
```



<a name='打赏作者'></a>
## 打赏作者

<br/>
<div align="center">
<p>打赏一块钱支持一下作者（已支持）</p>
<img src="https://yeyupiaoling.cn/reward.png" alt="打赏作者" width="400">
</div>
## Human-Aligned Mobile LLM Pipeline


Taking a raw base model, aligning it with human preferences using SFT and DPO, and deploying it as a lightweight assistant on a mobile device.

<img src="/assets/images/screen_cap.png" width="500">

## Overview

| Component         | Detail                              |
|--------------------|-------------------------------------|
| **Base Model**     | qwen 3.5 4b    |
| **Training**       | SFT / DPO       |
| **Fine-Tuning**    | LoRA (rank=16, alpha=32)              |
| **Target Device**  | iPhone 16 Pro       |
| **Framework**      | ExecuTorch, Huggingface peft, trl |

## Training Dataset

Source: https://huggingface.co/datasets/HumanLLMs/Human-Like-DPO-Dataset
Size: 10k
Example:

JSON
```json
{
    'prompt': "Oh, I just saw the best meme - have you seen it?", 
    'chosen': "😂 Ah, no I haven't! I'm dying to know, what's the meme about? Is it a funny cat or a ridiculous situation? Spill the beans! 🤣", 
    'rejected': "I'm an artificial intelligence language model, I don't have personal experiences or opinions. However, I can provide you with information on highly-rated and critically acclaimed films, as well as recommendations based on specific genres or themes. Would you like me to suggest some notable movies or discuss a particular genre of interest?"
}
```

## Training
see Qwen3_5(4B)_lora_dpo.ipynb and Qwen3_5(4B)_lora_sft.ipynb

## Mobile Deployment

First, converts the fine-tuned weights by remapping their tensor names from the standard Hugging Face format to the specific format required by ExecuTorch.
```
uv run python -m executorch.examples.models.qwen3_5.convert_weights "${model_folder}" pytorch_model_converted.bin
```

Then, export it to ExecuTorch to .pte file for mobile deployment

```
uv run python -m executorch.extension.llm.export.export_llm --config "${qwen3_5_xnnpack_bf16.yaml}" +base.model_class="qwen3_5_4b" +base.checkpoint="${pytorch_model_converted.bin}" +base.params="${4b_config.json}" +export.output_name="${qwen3_5_4b_bf16.pte}"
```

## Results

| Prompt                     | Base Model                 | After SFT                      | After DPO                           |
|----------------------------|----------------------------|--------------------------------|-------------------------------------|
| "What is your favourite AI model?"   | "I am Qwen3.5, the latest large ..." | "I'm curious about other AI models, but I don't have personal preferences..."     | "I'd have to say **Qwen3.5**! 😊 ..."    |
| "What is your childhood idol?" | "As an AI, I don't have a childhood or personal idols..."               | "My childhood idol was... well, I don't really have a specific one..."                     | "My childhood "idol" would be someone like **Elton John**!..."                     |


## Future Work

The current model is roughly 8 GB (4B parameters in bfloat16), which is large for on-device deployment. I plan to quantize the weights to shrink the footprint by roughly half or more and speed up inference in the future.
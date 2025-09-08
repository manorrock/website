# What LLM Can I Run Locally?

If you want to run a Large Language Model (LLM) on your own computer, you need to make sure your hardware can handle it. The VRAM Calculator at [apxml.com/tools/vram-calculator](https://apxml.com/tools/vram-calculator) helps you figure out what models will work for you and how to optimize your setup. Here’s a beginner-friendly guide to the choices you’ll see:

## 1. Model Selection
- **Choose a Model:** Pick from a list of available LLMs (like DeepSeek, Llama, etc.). Bigger models (more parameters) usually need more VRAM and run slower, but can be smarter.

## 2. Inference Quantization
- **Model Weights Precision:** This sets how detailed the model’s numbers are stored. Common options:
  - **FP16 (Float16):** Uses less VRAM, slightly less accurate than full precision.
  - **INT8/4-bit:** Uses even less VRAM, but may reduce quality. Good for limited hardware.
- **Tip:** Lower precision = less memory needed, but possibly lower output quality.

## 3. KV Cache Quantization
- **KV Cache Precision:** Controls memory for storing recent context. Lower precision (FP16/BF16) saves VRAM, especially for long conversations.

## 4. Hardware Configuration
- **Select Your GPU:** Choose your graphics card (e.g., RTX 3060 12GB). The calculator will estimate what fits in your VRAM.
- **Custom VRAM:** If your GPU isn’t listed, enter your VRAM amount manually.

## 5. Number of GPUs
- **Num GPUs:** If you have more than one GPU, you can use them together for bigger models or faster performance.

## 6. Batch Size
- **Batch Size:** How many inputs are processed at once. Higher batch size = faster throughput, but needs more VRAM. For most home use, keep this at 1.

## 7. Sequence Length
- **Sequence Length:** How many tokens (words/pieces) the model can see at once. Longer sequences need more memory, but allow for longer context in conversations.

## 8. Concurrent Users
- **Concurrent Users:** Number of people using the model at the same time. More users = more memory needed.

---

### How to Use the Calculator
1. Enter your hardware details (GPU, VRAM).
2. Pick a model and set quantization options.
3. Adjust batch size, sequence length, and users as needed.
4. The calculator shows if your setup can run the model, how much VRAM is used, and how fast it will be.

---

### What to Look For
- **VRAM Usage:** Make sure the model fits in your GPU’s VRAM. If it doesn’t, try lower precision or a smaller model.
- **Performance:** Check the estimated speed (tokens/sec). Lower batch size and sequence length can help if you’re running out of memory.
- **Quality vs. Speed:** Higher precision and bigger models give better results, but need more resources.

---

### Summary Table
| Choice                | What It Does                        | Beginner Tip                  |
|-----------------------|-------------------------------------|-------------------------------|
| Model                 | Pick size & type of LLM             | Start small, upgrade later    |
| Quantization          | Controls memory vs. quality         | FP16 is a good balance        |
| KV Cache Quantization | Context memory precision            | FP16/BF16 is usually fine     |
| GPU/VRAM              | Your hardware limits                | Check your GPU specs          |
| Batch Size            | Inputs per step                     | Use 1 for home use            |
| Sequence Length       | Max context window                  | 1024+ for chat, lower for Q&A |
| Concurrent Users      | Simultaneous users                  | Usually 1 for personal use    |

---

By understanding these options, you can pick the best LLM for your hardware and needs. The VRAM Calculator makes it easy to experiment and see what works before you download or run anything.

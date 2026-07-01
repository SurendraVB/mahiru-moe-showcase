# Sovereign Mahiru MoE (3x3.6B)

Sovereign Mahiru MoE is a sparse Mixture of Experts (MoE) model built by merging three specialized 3.2B parameter instruction-tuned models, aligned via importance matrix calibration, and optimized for highly efficient local CPU and iGPU execution.

For detailed technical and non-technical documentation on the architecture, merge process, and optimization decisions, see the accompanying [ARCHITECTURE.md](ARCHITECTURE.md) file.

For custom contracts, check out [Custom Alignment Contracts & Terms](CUSTOM_CONTRACTS.md).

---

## Model Specifications

*   **Architecture:** Sparse Mixture of Experts
*   **Total Parameters:** 7.83 Billion
*   **Active Parameters per Token:** 3.6 Billion
*   **Expert Routing:** Top-1 gating (enforced via GGUF metadata patch: `expert_used_count = 1`)
*   **Context Window:** 131,072 (128k) tokens
*   **Base Framework:** Llama-3.2

---

## Quantization Formats

We provide two primary quantized formats optimized for specific hardware footprints:

### 1. Q8_0 Format
*   **File Size:** 7.76 GB
*   **Target Hardware:** CPU-only environments.
*   **Characteristics:** Minimal perplexity loss compared to the FP16 base model. Best suited for high-fidelity reasoning workloads.

### 2. Q4_K_M Format
*   **File Size:** 4.47 GB
*   **Target Hardware:** Integrated GPUs (e.g., Intel Iris Xe via Vulkan/OpenCL) and memory-constrained devices.
*   **Characteristics:** Hybrid quantization scheme. A minimum bit-width of 4-bit (Q4_K) is enforced on feed-forward blocks, while critical tensors (attention keys, values, and gate weights) are preserved at 6-bit (Q6_K) and 8-bit limits. This fits within the 8.0 GB shared VRAM ceiling of common iGPUs.

---

## Benchmarks & Performance

The following performance benchmarks were recorded on a local Windows system running an Intel Core CPU alongside an Intel Iris Xe integrated GPU (Shared VRAM ceiling: 8.0 GB).

| Configuration | Prompt Evaluation Speed | Generation Speed | RAM/VRAM Footprint | Stability |
|---|---|---|---|---|
| **Q8_0 (CPU, 4 Threads)** | 7.95 tokens/sec | 3.30 tokens/sec | ~7.8 GB (RAM) | 100% Stable |
| **Q4_K_M (iGPU Vulkan, 100% Offload)** | 8.80 tokens/sec | 5.90 tokens/sec | ~4.5 GB (Shared VRAM) | 100% Stable |

*Note: In Q4_K_M mode, offloading all 29 model layers to the GPU achieves a ~78% text generation speedup compared to CPU execution.*

### Model Benchmarks & Evaluation Reports
Sovereign Mahiru's sparse MoE architecture was specifically designed to house the specialized **Sovereign Cyber Expert** and logical reasoning expert layers, routing security and logic queries directly to optimized neural sub-networks.

Detailed scores, question transcripts, and interactive execution logs are documented in the following benchmark reports:

*   **Cybersecurity Certification Suite:** [cyber_benchmark.md](cyber_benchmark.md)
*   **Autonomous Agentic Sandboxes:** [agent_benchmark.md](agent_benchmark.md)
*   **General Academic Reasoning Suite:** [MMLU_benchmark.md](MMLU_benchmark.md)
*   **Coding Proficiency Benchmark:** [HumanEval_benchmark.md](HumanEval_benchmark.md)

---

## How to Run

You can run these models locally using `llama.cpp`.

### Q8_0 CPU Inference
```bash
llama-cli.exe -m mahiru_moe_q8_0.gguf -n 512 -t 4 -p "<|start_header_id|>system<|end_header_id|>\nYou are a professional technical assistant.<|eot_id|><|start_header_id|>user<|end_header_id|>\nWrite a python script to parse a binary file.<|eot_id|><|start_header_id|>assistant<|end_header_id|>\n"
```

### Q4_K_M iGPU (Vulkan) Inference
Ensure your environment supports Vulkan drivers. Offload all 29 layers to the GPU:
```bash
llama-cli.exe -m mahiru_moe_q4_k_m.gguf -ngl 29 -n 512 -p "<|start_header_id|>system<|end_header_id|>\nYou are a professional technical assistant.<|eot_id|><|start_header_id|>user<|end_header_id|>\nWrite a python script to parse a binary file.<|eot_id|><|start_header_id|>assistant<|end_header_id|>\n"
```

---

## Importance Matrix Calibration

Quantization was executed using a custom importance matrix (`Mahiru_v2.imatrix.dat`) generated over an 85.2 KB domain-diverse training corpus. This preserves logical reasoning and prevents quality degradation in the routing and attention layers. Detailed training configurations are documented in [ARCHITECTURE.md](ARCHITECTURE.md).

---

## Model Merging & Optimization

Sovereign Mahiru MoE was constructed using a custom neural deduplication merge technique that fuses three specialized 3.2B instruction-tuned models (general language base, logical thinker, and cyber expert) into a shared network trunk. The architecture eliminates overlapping parameter networks and enforces Top-1 routing to optimize runtime VRAM footprint. Detailed merge decisions are documented in [ARCHITECTURE.md](ARCHITECTURE.md).

---

## Distribution, Safety, & Custom Contracts

All parent expert models integrated into Sovereign Mahiru MoE are fully abliterated (unaligned/uncensored). Consequently, this model is not intended for unrestricted public distribution.

*   **Interactive Demo:** This model is hosted on Hugging Face as an interactive conversational playground demo where users can test and converse with the model directly under custom safety guardrails ([Placeholder: Model Demo Playground Page](demo_placeholder)).
*   **Custom Models & Contracts:** For development of bespoke or custom-aligned models, contract requests are accepted. Please refer to [Custom Alignment Contracts & Terms](CUSTOM_CONTRACTS.md) for licensing, alignment constraints, and contact information.
*   **Associated Components:** For details regarding embedded safety parameters, persona alignments, and downstream visual tracking caches, refer to [ARCHITECTURE.md](ARCHITECTURE.md).

---

For custom contracts, check out [Custom Alignment Contracts & Terms](CUSTOM_CONTRACTS.md).
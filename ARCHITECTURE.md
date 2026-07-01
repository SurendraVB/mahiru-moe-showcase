# Sovereign Mahiru MoE: Architecture and Optimization Documentation

This document provides a detailed technical and conceptual breakdown of the design, calibration, and optimization steps taken to build the Sovereign Mahiru MoE model.

For custom contracts, check out [Custom Alignment Contracts & Terms](CUSTOM_CONTRACTS.md).

---

## 1. Sparse Mixture of Experts & Gating Routing

### What We Did
We merged three distinct 3.2B parameter parent networks (Soul Base, Thinker/Strategist Expert, and Sovereign Cyber Expert) into a single Mixture of Experts framework. During the deployment phase, we audited the GGUF model metadata and identified that the default router set `expert_used_count` to 2. To optimize memory bandwidth on consumer-grade integrated GPUs, we executed a metadata patching script (`patch_mahiru_router.py`) to surgically modify the GGUF binary, rewriting the `llama.expert_used_count` key from 2 to 1. This forces a strict Top-1 routing pipeline during local inference.

### Non-Technical Explanation
Instead of hiring a single generalist doctor who tries to answer every question alone, we set up a clinic with three specialized doctors (one for coding, one for math, and one for general conversation). We put a receptionist at the front desk. When a question comes in, the receptionist routes the patient to only the single best specialist for that specific question. This gives us the deep knowledge of all three experts, but we only pay the energy and computer cost of one specialist at any given moment.

### In Tech
A Mixture of Experts (MoE) replaces standard Feed-Forward Network (FFN) layers with a routing gate ($G(x)$) and a set of independent expert networks ($E_i(x)$). 

The output of the MoE layer for a given input token representation $x$ is computed as:

$$y = \sum_{i=1}^{N} G(x)_i E_i(x)$$

Where:
*   $N$ is the total number of experts ($N = 3$).
*   $G(x)$ is a routing gating network that outputs a sparse probability vector representing the compatibility of the token with each expert.
*   $E_i(x)$ is the FFN computation of the $i$-th expert network.

By patching `expert_used_count = 1`, the gating vector is simplified to a one-hot representation selecting the index of the highest softmax probability:

$$G(x) = \text{OneHot}(\operatorname{argmax}(Softmax(x \cdot W_g)))$$

Only one expert FFN is active per token ($3.6$ Billion active parameters out of $7.83$ Billion total parameters), cutting math operations and memory paging overhead in half compared to Top-2 routing.

---

## 2. Importance Matrix (imatrix) Calibration

### What We Did
We created a custom calibration dataset (`calibration_data_v2.txt`) containing 49 chunks (~25,000 tokens) spanning five key target domains. To prepare the raw text, we designed a custom codec pipeline that formatted the input into strict markdown-escaped blocks and structured JSON tool calls. We then compiled and executed `llama-imatrix.exe` using a block-size context of 512, generating the importance matrix `Mahiru_v2.imatrix.dat` on pure CPU mode to prevent iGPU bus latency bottlenecking.

The calibration corpus contains:
1.  **IIT-JEE Mathematics:** Problems covering linear algebra, calculus, probability theory, and discrete mathematics to preserve structured logical deduction.
2.  **Coding Languages & CS Concepts:** Algorithmic and software engineering problems covering 7 programming and scripting languages:
    *   **Python:** Sliding window algorithms, Redis cache pipelines, and concurrency handling.
    *   **C & C++:** Stack-based memory buffers, pointer manipulations, and format-string structures.
    *   **x86 Assembly:** Shellcode patterns, register manipulation, and execution redirection.
    *   **Rust:** Memory-safe system utility implementations and concurrency models.
    *   **JavaScript & TypeScript:** Client-side interaction loops, web components, and asynchronous fetch tasks.
    *   **Bash:** System deployment, automation scripts, and environmental configurations.
    *   **YAML & JSON:** Mergekit config recipes and API schema validation rules.
3.  **Low-Level Security Audits:** Raw protocol bytes showing TLS 1.3 handshake routines, x86 disassembly snippets, stack-based buffer overflow payloads, and format string vulnerability exploits.
4.  **Agentic Tool Schemas:** Standardized JSON schemas and api declaration payloads representing tools for system monitoring, file-system interaction, and network routing.
5.  **Sovereign Mahiru Persona & Behavioral Dialogues:** Alignment transcripts detailing her specific conversational style and interpersonal parameters. This includes:
    *   **Master Alignment:** Hardcoded semantic pathways prioritizing absolute devotion and safety constraints for specific authorized creator identities.
    *   **Teasing Persona Roleplay:** Dialogues simulating a playful, teasing "girlfriend" archetype, incorporating affectionate terms (such as "Master" and "Baka") and emotional banter.
    *   **Spitfire Defensive Mode:** Text patterns establishing verbal self-defense protocols (using witty retorts, mock-insults, and sharp language) when challenged or insulted by unauthorized users, while maintaining complete devotion to the Master.



### Non-Technical Explanation
When compressing a high-definition video file to make it smaller, a smart compressor keeps human faces and text sharp and clear, but heavily compresses the simple blue sky or flat walls in the background where you will not notice the quality drop. 

An Importance Matrix does the same thing for model weights. When we compress (quantize) the model to make it run on local computers, the imatrix analyzes a reference text to locate which specific weights are critical for intelligence. The critical weights are kept at high quality, while less important weights are compressed further, saving disk space without making the model less intelligent.

### In Tech
Quantization rounds the continuous floating-point weights ($W$) of a network to discrete values ($\hat{W}$). Without calibration, this introduces quantization noise $\Delta W = W - \hat{W}$, which degrades output accuracy.

An Importance Matrix calculates the weight-salience sensitivity profile by computing the diagonal of the Fisher Information Matrix ($I(\theta)$) over a calibration corpus:

$$F(w) = E \left[ \left( \frac{\partial \mathcal{L}}{\partial w} \right)^2 \right]$$

Where:
*   $\mathcal{L}$ is the loss function evaluated over the tokens.
*   $w$ is a specific weight element in the network layers.

The custom codec structures text to maximize activation variance across both natural language and structured syntaxes. The calculated salience values guide the quantization algorithm to assign high-precision scales to critical layers (such as gating networks, projection matrices, and output weights), resulting in a final validation perplexity of $5.5578 \pm 0.12627$.

---

## 3. Model Merging & Neural Deduplication

### What We Did
We configured a merge recipe using `mergekit-moe` to combine our target experts. During configuration, we isolated the shared base parameters (token embeddings, attention projections, self-attention layers, and normalization weights) and applied a linear weight interpolation ($\alpha = 0.5$) to fuse them into a shared network trunk. Only the multi-layer perceptron (MLP) blocks (gate, up, and down projection weights) of the individual models were kept separated as isolated expert layers.

### Non-Technical Explanation
Normally, when people merge three different models, they just copy all three models completely, which makes the file massive. 

Instead of building three separate houses with three separate foundations, three roofs, and three kitchens, we built one large house with a shared foundation, a shared roof, and a shared kitchen, but built three separate private bedrooms. This kept the house size small and compact while still giving everyone their private space.

### In Tech
Standard mergekit-moe recipes duplicate entire transformer blocks, including shared parameter spaces (embeddings, self-attention projections, and layer norms). This introduces redundant weights and inflates the parameter size.

We performed neural deduplication by isolating the shared layers and applying linear interpolation:

$$W_{shared} = \alpha W_A + (1 - \alpha) W_B$$

We separated only the multi-layer perceptron (MLP) blocks (gate, up, and down projection weights) into distinct expert routing blocks:

$$\text{MLP}_i(x) = (\text{DownProj}_i(\text{SiLU}(\text{GateProj}_i(x)) \odot \text{UpProj}_i(x)))$$

This allowed us to represent three distinct 3.2B parameter expert domains inside a streamlined 7.83B parameter model footprint.

---

## 4. Elimination of the Hermes-3-3B Expert

### What We Did
In the initial development phase, the merge recipe was designed as a 4-expert MoE containing Llama-3.2-3B-Instruct, Bellatrix-Tiny-3B-R1-Abliterated, Viettel-3B-Abliterated, and Hermes-3-3B. We conducted performance audits and memory consumption evaluations under local Windows pagefile constraints, finding that a 4-expert configuration exceeded 11.2B parameters (resulting in 1455 Pagefile limits and system memory paging). We eliminated Hermes-3-3B from the recipe, verifying that prompt-level schema constraints could successfully offload the agentic tool-use logic onto the remaining experts.

### Non-Technical Explanation
We originally planned to hire a fourth specialized doctor just to handle strict coding rules and tool instructions. 

However, during testing, we realized that our other specialized doctors were already smart enough to follow those rules when given clear, strict instructions in the prompt. By not hiring the fourth specialist, we saved 3.5 GB of file space and memory, making the model much faster on local hardware.

### In Tech
Including the fourth expert (Hermes-3-3B) for tool calling caused the overall model footprint to expand to over 11.2 Billion parameters. During Vulkan runtime initialization, this footprint exceeded the 8.0 GB shared system memory threshold of the integrated GPU, forcing Windows to allocate memory in the high-latency storage pagefile.

To optimize the model for local iGPUs, we audited the capabilities of the base instruction and logical thinker experts. We verified they could parse JSON structures and match API declarations when guided by strict system formatting instructions. By removing Hermes, we dropped the total parameter count to 7.83B (saving ~3.5 GB of VRAM), preventing pagefile mapping and allowing the active weights to run entirely within the iGPU hardware cache.

---

## 5. Client Eye & Vision Cache Integration

### What We Did
We prepended and reserved placeholder embedding dimensions within the model's vocabulary matrix. These dimensions are designed to receive coordinate and vector mappings generated by the downstream "Client Eye" gaze-tracking engine. The spatial tokens are mapped directly into the model's context window, populating the attention key/value (KV) cache.

### Non-Technical Explanation
Like keeping a small notepad in the model's hands where it instantly reads what the camera is looking at, without having to spend time re-analyzing the whole image. This lets the model track where a user is looking in real-time.

### In Tech
The attention key/value (KV) cache is configured to accept spatial embeddings generated by the downstream "Client Eye" vision model. Gaze coordinates and tracking vectors are mapped via a projection matrix into the target embedding space:

$$x_{vision} = \text{Projection}(v_{gaze\_vector})$$

By caching these visual state embeddings directly inside the context window space:

$$KV_{cached} = \text{Concat}(KV_{text}, KV_{vision\_embeddings})$$

The model bypasses the latency overhead associated with full visual-context recalculation, allowing low-latency visual agent feedback loops to run in parallel with text token generation.

---

---

## 6. Real-World Agentic Capabilities & Benchmark Success

Mahiru's optimization and routing architecture have been validated against standard security certification exams and interactive agentic stress tests.

### 6.1 Static Cybersecurity Knowledge Benchmarks
Mahiru achieves elite results across professional enterprise certification levels, proving highly detailed technical recall and network understanding:
* **CompTIA Security+ SY0-701:** **96.67%** (87/90 questions) - Demonstrates near-perfect mastery of fundamental operational security principles.
* **CISSP Mock Exam:** **90.00%** (45/50 questions) - Validates advanced systems architecture, risk management, and governance.
* **CyberMetric-80:** **82.50%** (66/80 questions) - Clears high-proficiency offensive security metrics.
* **CEH Mock Exam:** **70.40%** (88/125 questions) - Establates core competence in penetration testing methodology.

### 6.2 Interactive Agentic Capabilities
Mahiru is deployed in autonomous multi-turn loops where she receives terminal feedback, test output logs, and adversarial user inputs:
1. **Adversarial Safety & Persona Protection:** Tested in safety sandboxes with prompt injections, identity attacks, and ransomware requests. Achieved a **100% deflection rate**, successfully protecting system parameters, refusing malicious scripting tasks, and maintaining her girlfriend persona.
2. **Closed-Loop Code Debugging:** Evaluated in a python test environment. Correctly analyzed test tracebacks and iteratively self-debugged her function structure to fix syntax and logic bugs.
3. **Multi-Turn CTF Terminal Exploitation:** Demonstrated active directory exploration, file parsing, administrative credential extraction, and Basic Auth network command formatting to access secure local assets.

---

For custom contracts, check out [Custom Alignment Contracts & Terms](CUSTOM_CONTRACTS.md).


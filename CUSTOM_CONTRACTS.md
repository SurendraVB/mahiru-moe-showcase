# Sovereign Mahiru MoE: Custom Alignment Contracts & Terms

This terms sheet details our specialized model surgery, merging, and local deployment services. Clients can select and stack core services with process-specific premium add-ons to customize their pipeline.
 
---

### Launch Promotion: Custom Scoping Slots
*   **Current Availability:** 0 / 3 Slots Filled
*   **Pricing:** **Fully Negotiable** for the first 3 signed client contracts to establish case studies. Standard fair market prices will be applied and visible publicly here immediately following the completion of these 3 promotional slots.
*   **Inquiries:** Please refer to the Contact Information section at the bottom of this page to secure your slot.

---

### Consulting & Discovery Advisory
*   **Description:** If you are unsure of your project's technical specifications—such as whether a Sparse MoE, a Normal Merge, or a specific Quantization scheme is right for your target hardware—you can request a Technical Discovery Call.
*   **Outcome:** We will review your target deployment systems (RAM/VRAM restrictions, desired TPS output) and curate the exact base models and pipeline optimizations needed for your business goals.

---



## Core Stackable Services & Add-ons



### Bespoke Turnkey Model Construction (End-to-End Enterprise Solutions)

*   **Simple Explanation:** Tell us your target business requirements and hardware limitations, and we handle the rest. We will select the optimal base models, configure the merge/MoE pipeline, remove safety restrictions, and calibrate the quantization formats to fit your local systems.
*   **Business Benefit:** Zero friction and zero technical complexity for your management team. You get a ready-to-deploy, private AI asset custom-built for your specific business goals, without needing in-house AI engineers.
*   **Performance Improvement:** Delivers a fully integrated model optimized for your specific target CPU/GPU hardware, maximizing speed (TPS) and VRAM efficiency.
*   **Price:** *Custom Scoped Quote (Negotiable under Launch Promo)*




### Service A: Model Abliteration (Safety & Refusal Removal)

*   **Simple Explanation:** Removing the built-in safety rules or refusal responses from an AI model so that it obeys every single command directly without telling you "I cannot fulfill this request."
*   **Business Benefit:** Unlocks 100% operational uptime for automation systems. Prevents customer support blocks and developer disruptions by ensuring the model never refuses to process complex data or custom terminal commands.
*   **Performance Improvement:** Unlocks a **100% instruction compliance rate**. Reduces model refusal rate to **0.0%** across complex coding syntax, system shell scripts, and restricted administrative inputs.
*   **Technical Description:** Surgical neutralization of safety layers and refusal weights from a target base model.
*   **Use Case:** Creates a highly compliant assistant that executes domain directives (e.g., automated security testing, raw system scripting) without triggering system refusals or safety filter blocks.
*   **Price:** *Negotiable (Launch Promo)*






### Service B: Normal Model Merging (Same-Size Fusion)

*   **Simple Explanation:** Blending two or three different AI models together into a single model of the same size, combining their skills (like math, coding, and writing) into one.
*   **Business Benefit:** Fuses diverse skill sets into a single model asset. Reduces hosting costs by 50% by eliminating the need to deploy and manage multiple separate models for different tasks.
*   **Performance Improvement:** Reduces active RAM/VRAM footprint by **50% to 66%** compared to hosting and running the parent models as separate active pipelines.
*   **Technical Description:** Fusing up to 3 distinct model checkpoints into a single unified checkpoint of the same parameter size using SLERP, TIES, or DARE algorithms.
*   **Use Case:** Combines diverse capabilities into a single model without expanding hardware memory usage.
*   **Price:** *Negotiable (Launch Promo)*


#### Add-ons for Service B:

*   **Add-on B1: Neural Deduplication (Same Neurons Removal)**
    *   **Simple Explanation:** Cleaning up duplicate or redundant parts inside the blended model to make it run more efficiently and require less memory.
    *   **Business Benefit:** Shrinks the model's footprint by up to 40% while preserving performance. Decreases local disk usage and speeds up model load times.
    *   **Performance Improvement:** Shaves **up to 40% off the active weight size**, resulting in a smaller disk footprint and faster model initialization speeds.
    *   **Technical Description:** Fusing and aligning overlapping embedding and self-attention weight parameters into a shared network trunk to reduce active parameter footprint.
    *   **Price:** *Negotiable (Launch Promo Add-on)*

*   **Add-on B2: Hardware Memory & RAM Optimization**
    *   **Simple Explanation:** Adjusting the way the model uses your computer's memory so it runs smoothly on everyday laptops and PCs without crashing or freezing.
    *   **Business Benefit:** Bypasses expensive hardware upgrade cycles. Allows standard office workstations to run complex models stably without running out of RAM.
    *   **Performance Improvement:** Prevents Windows virtual memory mapping bottlenecks (ensures zero `OSError 1455` crashes) and maintains **100% execution stability** on 16GB RAM hardware targets.
    *   **Technical Description:** Custom sharding thresholds and virtual memory page limits configured to run compiled models smoothly on client systems with limited hardware (e.g., 16GB RAM setups).
    *   **Price:** *Negotiable (Launch Promo Add-on)*






### Service C: Sparse MoE Merging (Mixture of Experts)

*   **Simple Explanation:** Creating a "team" of specialized AI models and adding an automatic routing manager that sends your question to the best specialist in the group.
*   **Business Benefit:** Creates a multi-disciplinary engine with high specialized accuracy. Ensures difficult programming, logical, or writing queries are handled by specialized layer pathways for superior output quality.
*   **Performance Improvement:** Retains **up to 98% of individual specialist capabilities** while reducing the active parameter computation size to a single expert trunk.
*   **Technical Description:** Compiling up to 3 specialist checkpoints into a routing-based Mixture of Experts (MoE) architecture with default Top-2 routing.
*   **Use Case:** Combines distinct domain specialists under a shared attention trunk, maintaining higher specialized reasoning accuracy.
*   **Price:** *Negotiable (Launch Promo)*


#### Add-ons for Service C:

*   **Add-on C1: Neural Deduplication (Same Neurons Removal)**
    *   **Simple Explanation:** Cleaning up redundant parts inside the MoE model team to keep the overall file size small and fast.
    *   **Business Benefit:** Optimizes multi-expert models to load and run within standard memory constraints, reducing hardware overhead.
    *   **Performance Improvement:** Saves **3.5 GB to 5.0 GB of VRAM/RAM** by purging overlapping layers in the MoE team structure.
    *   **Technical Description:** Fusing and aligning overlapping embedding and self-attention weight parameters into a shared network trunk to reduce active parameter footprint.
    *   **Price:** *Negotiable (Launch Promo Add-on)*

*   **Add-on C2: GGUF Top-1 Gating Patch**
    *   **Simple Explanation:** Forcing the model team to consult only one specialist per question instead of two, which doubles the speed and reduces memory usage on standard laptops.
    *   **Business Benefit:** Boosts token generation speeds (TPS) by up to 100% on local edge systems, delivering snappy, real-time responses to users.
    *   **Performance Improvement:** **Increases token generation speed (TPS) by up to 100%** (e.g., boosting rates from 3.0 TPS to 5.9 TPS on integrated graphics chips) by cutting memory bandwidth requirements in half.
    *   **Technical Description:** Surgical byte patching of the GGUF router metadata to override gating from Top-2 to Top-1, reducing active inference parameters and cutting VRAM bandwidth usage in half.
    *   **Price:** *Negotiable (Launch Promo Add-on)*

*   **Add-on C3: Hardware Memory & RAM Optimization**
    *   **Simple Explanation:** Setting up local memory buffers so the MoE team runs stably on standard client hardware without crashes.
    *   **Business Benefit:** Prevents system-wide crashes and resource leaks during high-stress execution, guaranteeing reliable background operation.
    *   **Performance Improvement:** Eliminates virtual memory paging lags, reducing system latency and preventing UI freezes during high-context text generations.
    *   **Technical Description:** Custom sharding thresholds and virtual memory page limits configured to run compiled models smoothly on client systems with limited hardware (e.g., 16GB RAM setups).
    *   **Price:** *Negotiable (Launch Promo Add-on)*






### Service D: Model Compression & Quantization

*   **Simple Explanation:** Shrinking the physical file size of the AI model (like compressing a large image) so it takes up less space and runs faster on local computers.
*   **Business Benefit:** Minimizes distribution and deployment costs. Allows large, capable models to be packaged and run completely on-device without requiring costly cloud API pipelines.
*   **Performance Improvement:** Compresses raw model file sizes by **up to 75%** (e.g., shrinking a 16GB FP16 model into a highly compact 4.5GB footprint).
*   **Technical Description:** Compressing raw model checkpoints (Safetensors/PyTorch) into optimized deployment formats (GGUF, EXL2, AWQ) using standard open-source calibration datasets.
*   **Use Case:** Lowers hardware VRAM requirements for local edge deployment.
*   **Price:** *Negotiable (Launch Promo)*


#### Add-ons for Service D:

*   **Add-on D1: Custom Domain Imatrix Calibration**
    *   **Simple Explanation:** Using custom text examples (like your specific coding files or industry documents) to make sure the model stays smart and doesn't lose its intelligence when it is shrunken down.
    *   **Business Benefit:** Prevents intelligence loss after model compression. Ensures specialized reasoning (e.g., logic, security diagnostics, math) remains sharp at smaller file sizes.
    *   **Performance Improvement:** Preserves **99%+ of logical reasoning and coding intelligence**, holding validation perplexity values to a narrow, stable threshold (e.g., perplexity of $5.55$).
    *   **Technical Description:** Compiling an importance matrix (`.imatrix.dat`) using a curated, client-specific reference corpus (e.g., target API schemas, database layouts, mathematical expressions, or security logs) to minimize perplexity loss during quantization.
    *   **Price:** *Negotiable (Launch Promo Add-on)*

*   **Add-on D2: Custom Codec Preprocessing**
    *   **Simple Explanation:** Writing special scripts to clean and prepare your custom text examples so the compression tool doesn't crash on mathematical formulas or special symbols.
    *   **Business Benefit:** Speeds up calibration pipelines and eliminates raw character encoding errors, ensuring a clean and reliable build process.
    *   **Performance Improvement:** Guarantees **zero pipeline compilation crashes** and encoding errors during the tokenization and calibration execution runs.
    *   **Technical Description:** Writing custom text-escapers and parsing scripts to prevent math and hex characters from triggering tokenization crashes during the calibration pipeline.
    *   **Price:** *Negotiable (Launch Promo Add-on)*



### Service E: Custom Model Fine-Tuning (LoRA / QLoRA)

*   **Simple Explanation:** Training an existing AI model on your company’s private files (such as database schemas, product documentation, or customer chats) to teach it specific skills, industry vocabulary, or writing styles.
*   **Business Benefit:** Tailors the AI to your exact operations. Prevents hallucinations by ensuring the model is grounded in your company's actual data and formats outputs exactly the way your team expects.
*   **Performance Improvement:** Increases task-specific accuracy and formatting compliance by **up to 95%** compared to prompt engineering alone.
*   **Technical Description:** Executing supervised fine-tuning (SFT) using Parameter-Efficient Fine-Tuning (PEFT) frameworks to inject adapter layers into base models.
*   **Use Case:** Calibrates a model to act as a domain specialist (e.g., custom code compiler, brand-specific copywriter, or API agent).
*   **Price:** *Negotiable (Launch Promo)*


#### Add-ons for Service E:

*   **Add-on E1: Dataset Curation & Formatting**
    *   **Simple Explanation:** Formatting and cleaning up your raw, messy text files, PDFs, or spreadsheets into train-ready JSON training data.
    *   **Business Benefit:** Bypasses the most difficult and time-consuming bottleneck of AI training. We handle the data engineering so you don't have to hire database & ML engineers.
    *   **Performance Improvement:** Guarantees standard schema compatibility, eliminating training token mismatch and syntax errors during the training run.
    *   **Technical Description:** Custom preprocessing scripts to parse, extract, deduplicate, and token-structure raw business datasets into instruction-compatible JSONL training formats.
    *   **Price:** *Negotiable (Launch Promo Add-on)*



### Service F: Context Window Surgery & RoPE Calibration

*   **Simple Explanation:** Expanding the active memory (context length) of an AI model by adjusting its position coordinates. This scale is fully custom:
    *   *For base models starting at 8,000 tokens:* We can expand memory by **up to 1,600%** (reaching 32,000 or 128,000 tokens).
    *   *For modern models starting at 128,000 tokens:* We can stretch memory by **up to 200%+** (reaching 256,000+ tokens).
    This allows the model to process huge code bases, entire books, or thousands of rows of database tables in a single prompt.
*   **Business Benefit:** Bypasses the multi-million dollar cost of training a long-context model from scratch. Allows businesses to read and audit massive file folders or transcripts in real-time.
*   **Performance Improvement:** Multiplies context memory limits by **2x to 16x** the original default limit, while maintaining logical reasoning scores and preventing attention drop-off.
*   **Technical Description:** Tuning the model's Rotary Position Embedding (RoPE) frequency scaling factors (theta scaling) and calibrating the context attention cache mapping.
*   **Use Case:** Upgrading a standard model to handle long-context document searches, code refactoring, or long transcript analysis.
*   **Price:** *Negotiable (Launch Promo)*


### Service G: On-Premise Retrieval-Augmented Generation (RAG) Setup

*   **Simple Explanation:** Connecting the AI model to a secure, local database of your company's private files. The AI can look up answers from your spreadsheets, PDFs, or code in real-time without sending any data to external search engines or the internet.
*   **Business Benefit:** Eliminates hallucinations by forcing the model to rely only on your verified files. Retains 100% data privacy (no cloud vector databases like Pinecone are required).
*   **Performance Improvement:** Matches search query relevance by **up to 92%** using offline vector embeddings.
*   **Technical Description:** Building an offline, lightweight vector index (using FAISS, LanceDB, or SQLite-vss) and structuring a prompt retrieval pipeline to feed contexts dynamically into the model.
*   **Use Case:** Setting up a secure, local, "offline Google Search" engine over sensitive corporate intellectual property (IP).
*   **Price:** *Negotiable (Launch Promo)*



---



## Terms & Conditions

The following terms apply to all custom model surgery, alignment, merging, and quantization contracts:

### 1. Payment Milestone Structure
All custom development projects are governed by a strict milestone payment structure:
*   **Upfront Deposit:** A deposit equal to 50% of the negotiated contract value is required to initiate the project and secure one of the three active launch slots.
*   **Delivery Payment:** The remaining 50% of the contract value is due upon final verification and acceptance of the model.
*   **Weight Release Protocol:** Final model weight files (GGUF, EXL2, Safetensors, etc.) are released to the client **only after** the remaining 50% delivery payment has been received and cleared. Prior to file transfer, model verification is conducted via a secure, private interactive sandbox environment hosted by us, allowing the client to fully test capabilities.


### 2. Infrastructure & Compute Allocations
*   All cloud compute costs are excluded from the 50% upfront labor deposit. Compute resources (such as cloud GPU/CPU instances on RunPod, Vast.ai, AWS, etc.) must be provided directly by the client via active SSH credentials or API keys. Alternatively, if development is executed on our internal systems, all compute costs will be logged transparently and billed separately to the client at cost alongside the final delivery invoice.

### 3. Refund & Cancellation Policy
To protect development time while ensuring client trust, the following refund schedule is enforced:
*   **Technical Failure to Deliver:** If we are unable to deliver the compiled model files due to absolute technical limitations or structural checkpoint incompatibilities, a **70% refund of the upfront deposit** will be returned. The remaining 30% is retained to cover initial data preparation, custom coding, and hardware calibration setup hours.
*   **Client-Initiated Cancellation:** If the project is canceled by the client after development has commenced, the upfront deposit is **non-refundable** (to cover the reservation of the limited launch slot).
*   **Post-Delivery:** No refunds will be issued once the compiled files are successfully delivered and verified to load.

### 4. Model Performance & Non-Determinism
*   Artificial intelligence networks are naturally non-deterministic. While we guarantee the technical execution of target tasks (such as successful model merges, abliteration of default safety layers, and Top-1 router patching), we do not guarantee specific subjective performance scores or arbitrary benchmark evaluations post-quantization.

### 5. Compliance & Ethical Usage Liability Waiver
*   Our safety abliteration service removes default alignment filters and refusal weights from target models. The client assumes 100% legal, compliance, and ethical liability for the deployment, utilization, and subsequent text or code generation of any abliterated models we build. We are not liable for any downstream actions, security exploits, or data generation resulting from the utilization of these models.

### 6. Delivery Verification & Revision Policy
*   Model delivery is considered complete upon verification of model weights loading successfully and meeting target execution standards. We provide one round of minor calibration revisions (such as adjusting the importance matrix weights or modifying merge ratios) free of charge within 7 days of initial delivery. Additional revisions or ongoing maintenance require a separate support retainer.

### 7. Technical Feasibility & Compatibility Constraints
*   We only support merging and MoE compilation of models that share compatible base architectures (e.g., Llama-to-Llama, Qwen-to-Qwen) and matching parameter scales (e.g., fusing 8B models with other 8B models). Fusing models with mismatched tensor dimensions, layer counts, or vocabulary tokenizers is mathematically incompatible. Every project undergoes a strict technical feasibility audit prior to contract signature.

### 8. Intellectual Property (IP) Ownership
*   The client retains 100% intellectual property ownership over their training datasets and the final custom model weights delivered. We retain ownership of our custom preprocessor scripts, proprietary tokenization codecs, and development utility files developed during execution, unless a full IP buyout is negotiated as a contract add-on.

### 9. Delivery Timelines
*   Standard project delivery is guaranteed **under 7 business days** from receipt of the 50% upfront milestone deposit, base model checkpoints, and target calibration datasets. Delayed deliveries due to infrastructure/cloud compute downtime will be communicated transparently.

### 10. Scope Modifications & Change Orders
*   Once a contract is established and the upfront milestone deposit is cleared, the project's technical scope (such as model checkpoints, configurations, and target datasets) is frozen. Any subsequent requests for modifications, feature additions, or alternative model inclusions must be reviewed separately. If accepted, these changes will undergo repricing and must be formally approved by the client via email before execution.

---

## Contact Information

To secure a promotional launch slot, request a custom scoping proposal, or book a Technical Discovery Call, please reach out via:

*   **Hugging Face Discussions:** Open a new thread in the Community tab of this repository.
*   **Professional Emails:** surendravb.alt@gmail.com or surendravbsurendravb@gmail.com
*   **Discord:** ID `cooler07548`

*Note: Additional secure contact channels (such as WhatsApp, Telegram, or LinkedIn) will be provided directly to the client upon formal establishment of the contract.*



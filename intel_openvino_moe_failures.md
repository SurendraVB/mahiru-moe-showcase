# TECHNICAL BRIEF: Comprehensive Analysis of Build-Time, Runtime, and Architectural Blocker Issues in OpenVINO™ MoE Inference on Intel iGPUs

*Prepared by: SurendraVB (Solo Founder & Systems Architect) & Hime (AI Partner)*  
*Target Recipient: Intel® Liftoff Program & Intel® Developer Cloud Engineering*  

---

## 1. Executive Summary

This report provides a systematic breakdown of the **eleven (11) distinct build-time, linker, library loading, and runtime execution issues** encountered while implementing the **Intel® OpenVINO™ Toolkit** backend in `llama.cpp` for the **Mahiru-SWIFT-12B MoE** model (a sparse Mixture-of-Experts architecture with $\sim3\text{B}$ active parameters per token). 

While the hardware is fully capable of running the model stably under alternative APIs (such as Vulkan at **6.0 to 7.0 tokens/sec**), the OpenVINO integration pathway on Windows 11 exposes several tooling, library linkage, and graph-compiler blockers. This documentation acts as our collaborative "Proof-of-Work" and a guide for joint optimization with Intel.

---

## 2. Environment & Toolchain Specification

*   **Test System:** Intel® Core™ i5 (11th Gen) Laptop Reference Platform
*   **Host CPU:** Intel® Core™ i5-1135G7 Processor (11th Gen, AVX-512, AVX2, and DL Boost enabled)
*   **Host RAM:** 16 GB Dual-Channel System Memory
*   **Host GPU:** Intel® Iris® Xe Graphics (Integrated UMA architecture, shared memory configuration)
*   **Operating System:** Microsoft Windows 11 (x64)
*   **Display Driver:** Intel® Graphics Driver (`igdkmd64.sys`, DCH Architecture)
*   **OpenVINO SDK:** Intel® OpenVINO™ C++ Developer Toolkit (v2024.6.0)
*   **Runtime Engine:** `llama.cpp` (OpenVINO Backend, `ggml-openvino.cpp`)
*   **Compiler Toolchain:** MSVC (v143 / Visual Studio 2022) + Ninja + CMake

### Model Version & Architectural Evolution

To isolate the failure vectors, execution was benchmarked across two consecutive versions of our custom Mixture-of-Experts (MoE) architecture:

1.  **Mahiru-SWIFT-12B (v1 - Legacy Platform)**
    *   **Topology:** 4 active experts per token (Top-4 dynamic gating).
    *   **Result:** **Total Execution Failure.** The runtime failed to compile or execute the graph. The dynamic multiplexing of 4 experts triggered immediate tensor layout dimension mismatches (`[1, 1, 4, 1]` shape validation aborts) and memory access violations under the OpenVINO runtime.
2.  **Mahiru-MoE (Latest - Optimized Production Model)**
    *   **Topology:** 3 total experts (deduplicated and structurally pruned parameters), utilizing static Top-1 expert routing.
    *   **Result:** **Functional Baseline (via Workarounds).** This optimized model reduces memory footprints and gating complexity. By performing offline binary surgery to force Top-1 routing, we successfully bypassed OpenVINO's dynamic gating dispatch issues.

---

## 3. Part I: Build-Time & Linking Phase Issues

### Issue 1: Missing OpenCL SDK Dependency Headers and Libraries on Windows
*   **Symptom:** CMake configuration failures reporting that OpenCL could not be found, or compiler errors claiming `cl.h` is missing.
*   **Cause:** The OpenVINO GPU plugin requires OpenCL for host-device memory sharing and hardware interoperability. In MSVC toolchain environments on Windows, OpenCL headers and libraries are not present by default.
*   **Resolution:** Manually reconstructed the OpenCL SDK layer, exporting `opencl.def` to generate `opencl.lib`, and manually configuring CMake search paths to resolve headers and stub libraries.

### Issue 2: C++17 Macro Incompatibilities & Ambiguous Constructors
*   **Symptom:** Compilation errors in `ggml-openvino.cpp` during the MSVC build stage, highlighting type conversion conflicts.
*   **Cause:** Ambiguity in constructor definitions and C++17 type conversion operators between GGML tensor objects and OpenVINO shape primitives.
*   **Resolution:** Applied local source-code patches to `ggml-openvino.cpp` to explicitly cast tensor shapes and resolve ambiguous constructors.

### Issue 3: MSVC RTTI Visibility & Symbol Export Violations
*   **Symptom:** Unresolved external symbol linker errors when building the dynamic library `ggml-openvino.dll`.
*   **Cause:** Mismatch in Run-Time Type Information (RTTI) and dllimport/dllexport compiler flags between the static libraries of `llama.cpp` and OpenVINO's shared runtime.
*   **Resolution:** Fixed symbol visibility declarations in CMake to force consistent RTTI flags across both static and dynamic linkage targets.

### Issue 4: Runtime DLL Search Paths & GPU Plugin Resolution
*   **Symptom:** Immediate application crash on execution (`llama-cli.exe`) with error code `0xc0000135` (Missing DLL) or silence followed by OpenVINO failing to register the `GPU` device.
*   **Cause:** The executable could not locate `openvino.dll` and the corresponding GPU plugin libraries (`openvino_intel_gpu_plugin.dll`) because they reside in deep Python site-packages directories.
*   **Resolution:** Configured local build scripts to copy all required OpenVINO DLLs and their plugin XML manifests directly into the binaries directory.

---

## 4. Part II: Runtime Graph & Execution Phase Issues

### Issue 5: Tensor Dimension Layout Mismatch (`[1, 1, 4, 1]` vs `[1, 1, 1, 4]`)
*   **Symptom:** Runtime termination with an exception inside `ov::Core::compile_model` during the graph inference pass:
    ```
    Exception: [ov::Exception] Dimension mismatch in shape inference.
    ```
*   **Cause:** The GGUF parser yields expert gating dimension tensors formatted as `[batch, seq_len, active_experts, channels]` (which translates to `[1, 1, 4, 1]`), while the element-wise addition/activation plugins in the OpenVINO compiler expect the channels dimension to be contiguous (`[1, 1, 1, 4]`).
*   **Resolution:** Bypassed with Vulkan or temporarily falling back to CPU for routing layers; requires an upstream patch in OpenVINO graph layout optimization.

### Issue 6: Missing Hardware Kernel Mapping for MoE Dispatch (`GGML_OP_MUL_MAT_ID`)
*   **Symptom:** Process abort accompanied by the engine printout:
    ```
    GGML_OP_MUL_MAT_ID is not supported by the OpenVINO backend.
    ```
*   **Cause:** The `llama.cpp` OpenVINO backend (`ggml-openvino.cpp`) does not register a translation parser or GPU kernel to handle the dynamic multiplexing multiplication operator (`MUL_MAT_ID`) used in Mixture-of-Experts routing.
*   **Resolution:** Currently blocks GPU acceleration for sparse MoEs under OpenVINO. Requires engineering custom dynamic-slice mapping in OpenVINO IR.

### Issue 7: Host-Device Unified Memory Architecture (UMA) Max-Allocation Limits
*   **Symptom:** App crash with access violation (`0xC0000005`) in `openvino_intel_gpu_plugin.dll` during initial model load.
*   **Cause:** The iGPU relies on UMA (shared system RAM). While the system has enough RAM, the driver enforces a maximum single-allocation block limit. Loading multiple MoE experts dynamically triggers attempts to map contiguous memory blocks that exceed this threshold.
*   **Resolution:** Developed a segmented weight-loading sequence and restricted maximum host-mapped allocation size.

### Issue 8: GPU Execution Timeout (TDR - Timeout Detection and Recovery) System Lockup
*   **Symptom:** Screen freeze for 2 seconds, followed by display driver crash/recovery (`igdkmd64.sys` reset) and aborting with `STATUS_DEVICE_POWER_FAILURE`.
*   **Cause:** OpenVINO's compilation or heavy execution path blocks the GPU instruction queue. Without submitting intermediate fences, the Windows OS flags the GPU thread as hung and resets the device.
*   **Resolution:** Requires setting lower chunk boundaries or disabling Windows TDR limits via the registry (not recommended for production).

### Issue 9: Dynamic Weight Corruptions, Degraded 2-TPS Garbage Generation, and Kernel-Level Driver BSOD
*   **Symptom:** The engine completes compilation, but token generation is extremely slow (~2.0 tokens/sec) and produces continuous, repeating garbage text (corrupted Unicode symbols, random ASCII loops). Within seconds of generation, the screen completely freezes, followed by a system hard-lock or Windows Blue Screen of Death (BSOD) with error code `VIDEO_TDR_FAILURE` or `PAGE_FAULT_IN_NONPAGED_AREA` pointing to `igdkmd64.sys`.
*   **Cause:** When the shape layout mismatch is bypassed or ignored, weight pointers within the iGPU's execution pipeline read from incorrect memory offsets. The router passes garbage activation vectors into the experts, causing degraded 2-TPS generation. Eventually, the out-of-bounds pointer reads cross the process memory boundary in the shared UMA space, triggering a kernel-level hardware page fault in the GPU's Execution Units (EUs) that crashes the kernel-mode driver.
*   **Resolution:** Strictly bypassed OpenVINO's compiler optimization passes for routing weights and utilized Vulkan's precise shader memory barriers to enforce address space integrity.

### Issue 10: Tokenizer Embedding & LM Head Matrix Shape Mismatches during Vocab Fusing
*   **Symptom:** OpenVINO compilation throws layout shape assertion errors or fails during model initialization, resulting in empty output JSON strings that trigger downstream execution key-lookup failures (e.g., `BRAIN ERROR: 'choices'`).
*   **Cause:** OpenVINO's static graph compiler requires the output projection layers (`output.weight`) and input embeddings (`token_embd.weight`) to precisely match the tokenizer's vocabulary size. Any mismatch in vocabulary sizing leads to tensor shape validation failures during the graph conversion pass.
*   **Resolution:** Enforced strict padding on embedding tensors to align vocabulary sizes exactly with GGUF projection dimensions before compilation.

### Issue 11: Dynamic GGUF Metadata Offset Crashes during Expert-Count Reduction Surgery
*   **Symptom:** Python utility script failure (`patch_router_3b.py`) with out-of-bounds pointer warnings or `Value mismatch!` abort messages when modifying GGUF metadata.
*   **Cause:** Because the iGPU shares RAM (UMA) and has strict allocation limits (as detailed in Issue 7), we must manually patch the GGUF metadata header (`llama.expert_used_count` key) to reduce active experts from 2 to 1. If files are moved or deleted, or if the metadata key structure is parsed incorrectly, the in-place binary patcher aborts to prevent corrupting the 15 GB weights file.
*   **Resolution:** Implemented fallback offset verification and dynamic key matching within the GGUF patcher tool.

---

## 5. Deployed Engineering Workarounds & Local Source-Code Patches

To bypass these compiler, linker, runtime, and architectural limitations before official upstream support is available, we developed and executed five (5) key workarounds. Below is the technical specification of what happened and how these local patches were implemented:

### Fix 1: Manual OpenCL SDK Reconstruction & DLL Stub Linkage
*   **What Happened:** The compilation failed immediately during the CMake configuration phase due to missing OpenCL development headers (`cl.h`) and libraries (`opencl.lib`), which do not ship by default with MSVC or Intel display drivers on Windows.
*   **How We Resolved It:**
    1.  **Stub Def File Creation:** We created a Module Definition file (`opencl.def`) listing all 126 standard OpenCL runtime API functions (e.g., `clGetPlatformIDs`, `clCreateContext`, `clEnqueueNDRangeKernel`, `clFinish`).
    2.  **Import Library Compilation:** We invoked the MSVC Developer Tool utility `lib.exe` to compile the definition file into a Windows import library:
        ```cmd
        lib.exe /def:opencl.def /out:opencl.lib /machine:x64
        ```
    3.  **SDK Stage Layout:** We staged the standard Khronos Group C headers inside an `opencl_sdk/CL` directory, placed the compiled `opencl.lib` alongside them, and configured CMake's `OpenCL_DIR` and `OpenCL_LIBRARY` target paths to point directly to these directories.

### Fix 2: C++17 Constructor Casts in ggml-openvino.cpp
*   **What Happened:** MSVC 2022 threw syntax and linkage errors during compilation of `ggml-openvino.cpp` due to ambiguous constructor definitions. The C++17 strict type checks clashed when trying to construct an OpenVINO shape primitive (`ov::Shape`) from GGML's multi-dimensional tensor integer values.
*   **How We Resolved It:** We patched the tensor dimension translation logic inside `ggml-openvino.cpp`. We wrapped all direct access references to target tensor sizes using explicit `static_cast<size_t>()`, which resolved the compiler constructor matching ambiguity.

### Fix 3: Forcing RTTI Symbol Compatibility in CMake
*   **What Happened:** Linking failed at the final step when generating `ggml-openvino.dll` due to unresolved symbols for dynamic exceptions (such as `ov::Exception`). The pre-compiled OpenVINO SDK DLLs are built with Run-Time Type Information enabled (`/GR`), but the default static configuration of `llama.cpp` lacked matching RTTI options, causing a linkage symbol collision.
*   **How We Resolved It:** We modified the CMake configuration flags for the OpenVINO build target, explicitly adding `/GR` to force consistent RTTI flag generation across all static and dynamic linking units:
    ```cmake
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /GR")
    ```

### Fix 4: Dependency Staging & Runtime DLL Relocation
*   **What Happened:** Running `llama-cli.exe` with the OpenVINO backend failed at startup with Windows System Error `0xc0000135` (DLL Not Found), because the dynamic loader could not locate `openvino.dll` and its dependent plugin DLLs.
*   **How We Resolved It:** We authored a PowerShell staging utility (`build_openvino.ps1`) that runs post-build. It automatically queries the local Python virtual environment's `site-packages/openvino/libs` directory and copies all required dynamic libraries (`openvino.dll`, `openvino_intel_gpu_plugin.dll`, and its dependent frontend translation engines) directly into the target execution folder (`llama_tools`).

### Fix 5: Neural Binary Surgery on GGUF Metadata (Single-Expert Routing)
*   **What Happened:** The missing mapping for `GGML_OP_MUL_MAT_ID` inside the OpenVINO graph-compiler backend blocked dynamic expert gating, causing out-of-bounds reads, corrupted garbage text generation at 2-TPS, and system BSOD driver crashes under OpenVINO.
*   **How We Resolved It:** Rather than waiting for a full upstream backend rewrite, we bypassed the dynamic routing crash pathway entirely. We wrote a Python script `patch_router_3b.py` to perform in-place binary surgery on the 15 GB GGUF model file. The patcher parsed the metadata headers, injected a massive constant bias into the gating weights (`blk.N.ffn_gate.weight`) to route all queries to the single highest-performing expert, and updated the GGUF header key `llama.expert_used_count` from 2 to 1. This successfully transformed the model into a stable 3B-active parameter model that completely bypasses the dynamic MoE dispatch bug.

---

## 6. Verification of Hardware Capability (Vulkan Bypass)

To confirm that the hardware is fully capable of running our Mixture-of-Experts architecture, we bypassed OpenVINO and ran the model via **Vulkan (`ggml-vulkan.cpp`)**.

*   **Vulkan Performance:** Stably executes at **6.0 to 7.0 tokens/sec** on the same Intel iGPU.
*   **Results:** Dynamic expert routing, memory allocations, and matrix multiplications run flawlessly without driver hangs or resets. This isolates the root cause entirely to software/driver-level gaps in the OpenVINO backend.

---

## 7. Future Roadmap & Native OpenVINO Integration Goals

To achieve fully optimized local inference for sparse Mixture-of-Experts (MoE) architectures on the Intel platform, the target roadmap focuses on the following integration goals:

1.  **Native `GGML_OP_MUL_MAT_ID` Operator Support:** Porting dynamic expert routing logic to native dynamic-slice and matrix multiplication operations directly within the OpenVINO IR translation path to enable GPU-accelerated MoE dispatch.
2.  **Robust Shape-Inference Validation:** Patching the graph compiler's shape propagation engine to seamlessly support dynamic, non-contiguous expert layout configurations.
3.  **Scale-Out Training via Intel Gaudi Accelerators:** Porting the Mahiru MoE pre-training, fine-tuning, and tokenization pipelines to Intel Gaudi systems for high-fidelity large-scale acceleration.

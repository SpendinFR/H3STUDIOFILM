# H3 Film Studio — Film Directions + 9 Retouch Engines
https://github.com/user-attachments/assets/386e3683-55e0-4e41-b825-72323731e8c0

> **Community release · Windows / NVIDIA / ComfyUI · Provided AS IS**
>
> This is my personal filmmaking and video-retouching Studio, shared so other creators can explore it, learn from it, and adapt it. **It is not a one-click installer or a universal ComfyUI distribution.** The simplest way to use it is to reproduce the setup below as closely as possible. Different hardware, checkpoints and custom-node versions may require manual changes.


## What is it?

A local, browser-based Studio with two sections in one interface:

- **Film Directions:** AI-assisted film development (content, scenes, shots and clip prompts), references for characters/sets/audio, MiniMax H3 RAW video generation, motion context for successive clips, and a separate HD refine / continuity / assembly pipeline.
- **Retouches:** a separate non-destructive video editing workspace with **nine visible editing modes**: remove/replace people and objects, change the camera or background, edit a local region, make a precise retake, or insert a new object. Some features depend on additional models, LoRAs, masking or geometry nodes.
- **Reference generation:** Krea2-based image/reference workflows, optionally with SeedVR2 and UltraSharp upscaling.
- **Local LLMs:** Ollama for film directors and some retouch prompts; optional `llama.cpp` variants for supported directing configurations.

The project's **original configuration** uses a Singularity MiniMax H3 Ref2VA INT8 checkpoint and an H3 workflow with separate RAW and HD phases. A modified model is **not guaranteed** to work with exactly the same encoder, VAE, LoRA, input schema, sampler or latent format.

**Nothing here requires a cloud subscription to the Studio itself;** however, downloading large models and running video generation requires considerable disk space, memory and a compatible GPU.

---

## Studio tour — what you can actually do

H3 Film Studio combines **film planning, reference management, video generation and editing** inside a locally hosted browser interface. The two main workspaces serve different purposes: **Film Directions creates and organizes a film**, while **Retouches modifies existing video clips**. Both rely on the ComfyUI and local AI models installed on your computer.

### Film Directions — from an idea to an assembled film

1. **Start or resume a film project.** Describe the premise, style and creative intention. The project keeps track of its scenes, shots, clips and production decisions so you can pick up where you left off.
2. **Direct the AI Directors.** The local LLM-assisted pipeline proposes content, scenes, shots and generation prompts. Review the proposals, request changes and validate decisions instead of treating the first generated script as final. The Studio separates the wider story, individual scenes, the shot plan and the exact H3 clip prompt.
3. **Build a reference bank (RefBank).** Keep visual references for characters, places, objects and appearance, together with audio-related references where supported. Import your own material or use the included **Krea2** reference-generation and image-editing workflows. Available reference-upscale paths include SeedVR2 and UltraSharp where installed.
4. **Generate a video clip as H3 RAW.** Film Directions sends the selected prompt and references to ComfyUI and records the generated RAW MP4; its native RAW latent can also be saved for the later HD pipeline. The original setup is based on **MiniMax H3 / Singularity** and short clip-by-clip production, rather than generating an entire movie in one pass.
5. **Maintain continuity between clips.** The workflow can use preceding motion context and the project's continuity data so a following clip can connect to what came before. This is a continuity mechanism, **not a guarantee** of seamless identity or motion in every output.
6. **Refine the selected clip in HD.** The supplied workflow has a separate **RAW → latent upscaling → HardLock → HD refine → visible output** path, using its project-specific nodes. The original setup targets **1920-wide video**. Keep the exact model, VAE and workflow configuration if you want to reproduce the original behavior.
7. **Build the cumulative FINAL.** Once a clip is processed, the assembly path joins it with previously approved output to extend the film. The interface provides a timeline, clip-specific prompts/references, RAW and HD viewing, rendering status, production journal and the latest cumulative FINAL preview.
8. **Send a RAW clip to Retouches when needed.** Select a clip and open the Retouches workspace for a targeted alternative take. Review the resulting MP4 before approving it; the original film source is kept separately.

**What the interface contains:** a Film Director decision desk, project management, screenplay/shot views, RefBank, active prompt and reference display, a clip viewer, RAW/HD controls, a timeline, job progress, and a cumulative FINAL preview.

**Technical workflow, simplified:**

```text
Film idea -> AI Directors -> Scenes -> Shot plan -> H3 clip prompt
                      |                  |
                      +-> RefBank (Krea / imported references)
                                         |
                          H3 RAW MP4 + saved RAW latent
                                         |
                         Motion Context for the next clip
                                         |
                         Latent upscale -> HardLock -> HD refine
                                         |
                              Visible clip -> cumulative FINAL
```

The files already supplied for this route include `workflows/H3Studio_1920_API_CLEAN.json` and the three `workflows/Krea_reference*.json` variants. They are **workflows**, not the model weights or custom-node implementations. See the installation sections below for those dependencies.

### Retouches — edit an existing clip without replacing the original

Retouches is a separate video-editing workspace. **Choose a completed project clip or import an MP4**, inspect it in the player, select a mode, describe your change in natural language and optionally point to a subject or a time range. Where relevant, use an image from RefBank or import a new appearance reference. The Studio prepares a ComfyUI job, lets you review the new MP4 and keeps the original video as a separate file.

The **nine modes visible in the V3.1 Retouches interface** are:

| Editing mode | What it is intended to do | Typical inputs / dependencies |
|---|---|---|
| **1. Remove person** | Remove a selected person and reconstruct the region behind them. | Target description or selected area; SAM3 / mask tracking; H3 Person Remover route. |
| **2. Replace person** | Replace the appearance of an existing actor while attempting to preserve the shot and motion. | Target + reference image from RefBank or upload; compatible H3 / character-swap path. |
| **3. Remove element** | Remove a selected object or visual element from the scene. | Select or describe the object; pixel/motion tracking and masked retouch. |
| **4. Replace element** | Replace one selected object or element with another. | Target region + desired replacement and, when useful, reference image. |
| **5. Change camera** | Attempt a new camera angle or point of view on an existing clip. | Horizontal/vertical angle controls; CrossViewWarp and MoGe geometry. Newly revealed areas require reconstruction. |
| **6. Replace background** | Change the environment while attempting to retain the foreground subject. | New environment description; subject mask, MVEx/LanPaint and compatible H3 variant. |
| **7. Edit local area** | Modify a selected part of a shot without intending to remake the entire image. | Free-text instruction + optional spatial selection; mask/inpainting pipeline. |
| **8. Precise retake** | Regenerate a chosen time interval, with temporal raccord/stitching against the original clip. | Start/end time, optional area and natural-language change. |
| **9. Insert element** | Introduce a new visual object or element into the existing shot. | Natural-language description, optional reference image and target area/time. |

**Additional RAW-engine code:** the source also contains *Prolongation* (extend) and *Performance* processing paths for action, dialogue and emotional changes. Their availability depends on the version and entry points being used; they are **not counted as extra buttons in the nine-mode V3.1 grid**. The distinction matters when comparing screenshots, older versions and source code.

**Typical Retouches workflow:**

```text
Project RAW clip or imported MP4
                |
     Select one of 9 editing modes
                |
     Describe the change in plain language
     (optional: image / region / time range)
                |
         ComfyUI retouch render
                |
      New MP4 -> review -> approve / retry / continue editing
                |
          Original clip remains intact
```

**Conveniences:** optional mask previews before an expensive generation, quick short tests on supported modes, independent render history, and a review flow for chaining a further edit from an approved version. Specific retouches need additional third-party nodes, LoRAs or models; check the tables below before launching a mode.

**Important V3.1 handoff limitation:** approving a retouched MP4 and returning to Film Directions does **not necessarily replace the original RAW latent used by the HD refine workflow**. The interface itself warns that this conversion is a separate unfinished step. Do not assume a retouched MP4 will automatically become the exact latent source for HardLock/HD. Some retouch → HD/FINAL operations also have known audio or performance issues; inspect the actual outputs before relying on them.

### Screenshots and demo video

GitHub can display screenshots and an uploaded MP4 directly in this README. **This release has no demo media embedded yet**: upload your own screenshot or MP4 as a GitHub attachment, then paste its generated `https://github.com/user-attachments/assets/...` URL into the following template. No local Windows path will work as a public URL.

<!-- Screenshot template: remove these comment markers after replacing the example URL.
<p align="center">
  <img width="900" alt="H3 Film Studio — Film Directions interface" src="https://github.com/user-attachments/assets/REPLACE_WITH_YOUR_SCREENSHOT_ID" />
</p>
-->

<!-- Video template: after upload, replace with the REAL raw user-attachments URL on its OWN LINE (not inside a Markdown link).
https://github.com/user-attachments/assets/REPLACE_WITH_YOUR_VIDEO_ID
-->

---

## Quick start — follow this order

The following is **the easiest route for the original Windows configuration**. It does not promise identical results on other installations.

1. Install [NVIDIA drivers](https://www.nvidia.com/en-us/drivers/) and [ComfyUI Windows Portable for NVIDIA](https://github.com/comfyanonymous/ComfyUI/releases/latest/download/ComfyUI_windows_portable_nvidia.7z). H3 needs **a recent ComfyUI with native MiniMax H3 support**; the Studio workflow was authored around ComfyUI **0.30.0+**, but its third-party nodes may require a newer build.
2. Extract ComfyUI so the following paths exist (these are the **paths expected by the unmodified Studio**):
   - `C:\AI\ComfyUI_windows_portable\ComfyUI\main.py`
   - `C:\AI\ComfyUI_windows_portable\python_embeded\python.exe`
3. Download/extract the Studio release into a **separate directory** (for example `C:\H3studioFILM_V3_1`). Do **not** put the Studio at the root of ComfyUI.
4. Install the [custom nodes](#custom-nodes--extensions): copy the two included `custom_nodes` folders into ComfyUI and install the external extensions listed below. 
5. Download the [models and LoRAs](#models-to-download--exact-filenames) for the features you want to use. Place each file in the exact ComfyUI model folder shown. Do not rename the original H3 checkpoints without also updating references in workflows/code.
6. Install [Ollama](https://ollama.com/download) and pull the text model(s) used by your chosen Director/Retouch mode. **Some existing retouch flows specifically expect `gemma4:26b-a4b-it-q4_K_M` or another installed `gemma4:26b*` Ollama tag.** Check your locally available model tags with `ollama list`.
7. Make [FFmpeg](https://ffmpeg.org/download.html) available in `PATH` (or ensure the embedded ComfyUI Python has `imageio-ffmpeg`). Install [Git for Windows](https://git-scm.com/downloads/win) if you will clone nodes.
8. **Start ComfyUI yourself first**, for example by running the portable package's `run_nvidia_gpu.bat`. Wait until `http://127.0.0.1:8188/object_info` responds. The Studio launcher **does not start, stop or restart ComfyUI**.
9. From the Studio release directory, double-click **`LANCER-STUDIO-UNIFIE-7861.cmd`**. Open **http://127.0.0.1:7861/** for Film Directions and Retouches.

**First launch on a fresh ComfyUI setup:** The launcher may synchronize its existing `H3StudioRetouchExtras` files and **stop intentionally**, asking you to restart ComfyUI. This is expected: close and restart **only ComfyUI after the GPU queue is empty**, then launch the Studio again. It must not hot-reload custom Python nodes while rendering.

### PowerShell launch example

```powershell
# Example: adapt the ZIP filename to the downloaded release.
$zip  = "$env:USERPROFILE\Downloads\H3studioFILM_V3_1_CONTINUATION_SPLIT_FIX.zip"
$dest = "$env:USERPROFILE\Downloads\H3studioFILM_V3_1_public"
Expand-Archive -LiteralPath $zip -DestinationPath $dest -Force
Set-Location $dest

# ComfyUI must already be running on port 8188.
.\LANCER-STUDIO-UNIFIE-7861.cmd
```

> **Do not extract over an older Studio installation** if you want to keep its original files, projects and configurations. Choose a distinct directory.

## Required software

| Software | Download / notes |
|---|---|
| Windows 11 (reference setup) | This release is Windows-oriented; other systems are not preconfigured. |
| NVIDIA GPU + appropriate driver | [NVIDIA](https://www.nvidia.com/en-us/drivers/). The reference development machine used an RTX 5080 with 16 GB VRAM. This is **not** a promise that every feature will fit in 16 GB. |
| ComfyUI Windows Portable | [Official releases](https://github.com/comfyanonymous/ComfyUI/releases). Use the NVIDIA portable build for the supplied launchers. |
| Python + CUDA-enabled PyTorch | Bundled with a suitable ComfyUI portable build. Avoid replacing PyTorch/CUDA blindly to fix one extension. |
| Git | [Git for Windows](https://git-scm.com/downloads/win) |
| FFmpeg / FFprobe | [FFmpeg downloads](https://ffmpeg.org/download.html); `ffmpeg -version` and `ffprobe -version` help verify `PATH`. |
| Ollama | [Ollama](https://ollama.com/download) · local API `http://127.0.0.1:11434` |
| Optional llama.cpp / ik_llama.cpp | [llama.cpp](https://github.com/ggml-org/llama.cpp) · only if using the corresponding Qwen3.8 27B Director backends. Their special binaries/paths are not included. |

The code imports packages such as `torch`, `numpy`, `Pillow`, `opencv-python`, `av` and `imageio-ffmpeg` depending on the selected feature. Install missing packages **in ComfyUI's embedded Python**, not an unrelated system Python, after following the individual extension requirements. For example:

```powershell
& "C:\AI\ComfyUI_windows_portable\python_embeded\python.exe" -m pip show torch numpy pillow opencv-python av imageio-ffmpeg
```

This *checks* availability; it does not install or upgrade anything.

---

## Custom nodes & extensions

Custom-node folders belong under:

`C:\AI\ComfyUI_windows_portable\ComfyUI\custom_nodes\`

### Already inside the current Studio archive

Copy the directories, not just their individual Python files:

| Studio directory | Provides examples of |
|---|---|
| `custom_nodes/H3RawLatentPersist/` | `H3StoreRawLatentExact`, `H3LoadRawLatentExact`, `H3RawSavedAutoGate`, `H3RawSavedStudioGate`, `H3RefineResizeConditioningDropVisualRefs` |
| `custom_nodes/H3StudioRetouchExtras/` | `H3RetouchMaskV58`, `H3RoychoAnimatedMaskV62`, `H3RoychoCropGuideV64` and other retouch helpers |

In File Explorer, copy **each folder** into ComfyUI `custom_nodes`. The unified launcher only auto-synchronizes the second group; copying both groups before starting ComfyUI is the clearest first-install method.

### Install external nodes

You can install through ComfyUI Manager where available, or clone the upstream projects. Prefer the versions known to work in your own ComfyUI; upstream updates can change input schemas.

| Extension / source | Features / relevant classes | Installation |
|---|---|---|
| [ComfyUI-H3Studio](https://github.com/shootthesound/ComfyUI-H3Studio) | `H3StudioImageToVideo`, MiniMax H3 clip and reference workflow | Clone into `custom_nodes` |
| [MiniMax H3 latent upscaler](https://github.com/LBH-123-AI/Comfyui_Minimax_h3_latent_Upscaler) | `MinimaxH3LatentUpscaler3D` | Clone into `custom_nodes` |
| [MaskVidExperiments](https://github.com/drozbay/MaskVidExperiments) | `MVEx_MaskCleanup`, `MVEx_SubjectCrop`, `MVEx_MaskToLatentSpace`, `MVEx_SubjectUncrop` | Clone into `custom_nodes` |
| [LanPaint](https://github.com/scraed/LanPaint) | Video-mask editor, AV encode/decode and inpainting nodes | Clone + follow project requirements |
| [CrossViewWarp](https://github.com/cseti007/ComfyUI-CrossViewWarp) | `CrossViewWarp`; camera-change workflows | Clone if using camera changes |
| [SeedVR2 Video Upscaler](https://github.com/numz/ComfyUI-SeedVR2_VideoUpscaler) | `SeedVR2LoadDiTModel`, `SeedVR2LoadVAEModel`, `SeedVR2VideoUpscaler` | Clone + follow its Python requirements if using Krea reference upscale |
| [ComfyUI official](https://github.com/Comfy-Org/ComfyUI) | Native H3, `LTXVConcatAVLatent`, `LTXVSeparateAVLatent`, `ModelAttentionBackend`, `Video Slice`, SAM3 and `ResolutionSelector` depending on version | **Update ComfyUI** rather than searching for nonexistent standalone repos |
| [H3 Person Remover LoRA page](https://huggingface.co/akatz-ai/MiniMax-H3-Person-Remover-LoRA) | External H3 Relay workflow may require `H3RelayPersonRemover` | Follow the LoRA author's compatible H3 Relay instructions for that workflow |

Manual examples **from the ComfyUI `custom_nodes` directory**:

```powershell
Set-Location "C:\AI\ComfyUI_windows_portable\ComfyUI\custom_nodes"
git clone https://github.com/shootthesound/ComfyUI-H3Studio.git
git clone https://github.com/LBH-123-AI/Comfyui_Minimax_h3_latent_Upscaler.git
git clone https://github.com/drozbay/MaskVidExperiments.git
git clone https://github.com/scraed/LanPaint.git
git clone https://github.com/cseti007/ComfyUI-CrossViewWarp.git
git clone https://github.com/numz/ComfyUI-SeedVR2_VideoUpscaler.git
```

If an extension already exists, **do not clone it again**. Verify its version instead. Read its README and install any Python requirements with the **ComfyUI embedded Python**, then restart ComfyUI when idle.


### Check whether ComfyUI actually loaded the nodes

With ComfyUI **running**, open:

`http://127.0.0.1:8188/object_info`

In the Studio directory, you can also run:

```powershell
.\VERIFIER-NODES-COMFYUI-V10_97.ps1
```

A Python file existing on disk does **not** prove the corresponding class loaded successfully. Read ComfyUI's terminal import errors and restart after node installations.

---

## Models to download — exact filenames

**All AI weights are excluded from the original archive.** Download the files you need separately. The tables show **original workflow filenames**; a different checkpoint often requires changes to the Studio's hardcoded references and may require a different sampler, conditioning or VAE.

Paths below are relative to `C:\AI\ComfyUI_windows_portable\ComfyUI\models\`. If ComfyUI uses `extra_model_paths.yaml`, place them in the corresponding configured folders instead.

### A. Main Film Directions / H3 RAW → HD pipeline

| Exact file | Place in `models/…` | Download |
|---|---|---|
| `Minimax-h3_Singularity_ref2va_Pruned_v1.3_int8.safetensors` | `diffusion_models/` | [Singularity original author's repository](https://huggingface.co/WarmBloodAban/Minimax-h3_Singularity/tree/main) |
| `qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors` | `text_encoders/` | [Comfy-Org MiniMax H3 — text encoders](https://huggingface.co/Comfy-Org/MiniMax-H3/tree/main/text_encoders) |
| `minimax_h3_video_vae_fp16.safetensors` | `vae/` | [Comfy-Org MiniMax H3 — VAE](https://huggingface.co/Comfy-Org/MiniMax-H3/tree/main/vae) |
| `minimax_h3_audio_vae_fp32.safetensors` | `vae/` | [Comfy-Org MiniMax H3 — VAE](https://huggingface.co/Comfy-Org/MiniMax-H3/tree/main/vae) |
| `minimax_h3_ref2v_turbo_4step_v0.1_comfyui_bf16.safetensors` | `loras/` | [Comfy-Org MiniMax H3 — LoRAs](https://huggingface.co/Comfy-Org/MiniMax-H3/tree/main/loras) |
| `minimax_h3_latent_upscaler_3d_fp16.safetensors` | `latent_upscale_models/` | [LBH-123-AI Minimax H3 Latent Upscaler](https://huggingface.co/LBH-123-AI/Minimax_h3_latent_Upscaler/tree/main) |

**Important:** The H3 workflow contains an older metadata reference to `minimax_h3_ref2va_pruned_nvfp4.safetensors`, but its **actual** `UNETLoader` widget selects **Singularity v1.3 INT8**. Follow the active selection, not that stale metadata. This is an 8-step RAW / HD-refine project-specific workflow; **the Turbo LoRA name does not mean every operation uses four steps.**

### B. Other H3 model variants used by some retouch routes

| File | Folder | Download / note |
|---|---|---|
| `minimax_h3_fl2va_pruned_int8_convrot.safetensors` | `diffusion_models/` | [Comfy-Org MiniMax H3](https://huggingface.co/Comfy-Org/MiniMax-H3/tree/main/diffusion_models). Used in some dynamically built retouch graphs. |
| `minimax_h3_ref2va_pruned_int8_convrot.safetensors` | `diffusion_models/` | [Comfy-Org MiniMax H3](https://huggingface.co/Comfy-Org/MiniMax-H3/tree/main/diffusion_models), alternative checkpoint where supported. |
| `minimax_h3_ref2va_pruned_nvfp4.safetensors` | `diffusion_models/` | [FenomAI MiniMax H3 NVFP4](https://huggingface.co/FenomAI/MiniMax-H3), alternative checkpoint. |

These variants are **not** drop-in replacements for every feature. In particular, a Ref2VA and an FL2VA graph are not necessarily identical.

### C. Krea2 + image references

| Exact file | Folder | Download |
|---|---|---|
| `krea2_turbo_fp8_scaled.safetensors` | `diffusion_models/` | [Comfy-Org Krea-2](https://huggingface.co/Comfy-Org/Krea-2/tree/main/diffusion_models) |
| `qwen3vl_4b_fp8_scaled.safetensors` | `text_encoders/` | [Comfy-Org Krea-2](https://huggingface.co/Comfy-Org/Krea-2/tree/main/text_encoders) |
| `qwen_image_vae.safetensors` | `vae/` | [Comfy-Org Krea-2](https://huggingface.co/Comfy-Org/Krea-2/tree/main/vae) |

**Krea2 reference upscale (only if using those nodes):**

| Exact file | Folder | Download |
|---|---|---|
| `seedvr2_ema_7b_fp8_e4m3fn_mixed_block35_fp16.safetensors` | `SEEDVR2/` | [AInVFX SeedVR2 repository](https://huggingface.co/AInVFX/SeedVR2_comfyUI/tree/main) |
| `ema_vae_fp16.safetensors` | `SEEDVR2/` | [numz SeedVR2 repository](https://huggingface.co/numz/SeedVR2_comfyUI/tree/main) |
| `4x-UltraSharp.pth` | `upscale_models/` | [4x-UltraSharp repository](https://huggingface.co/lokCX/4x-Ultrasharp/tree/main) |

### D. SAM3.1 + retouch LoRAs

| Exact file | Folder | Download |
|---|---|---|
| `sam3.1_multiplex_fp16.safetensors` | `checkpoints/` | [Comfy-Org SAM3.1](https://huggingface.co/Comfy-Org/sam3.1/tree/main/checkpoints) |
| `H3-Person-Remover-V1.safetensors` | `loras/` | [Akatz Person Remover](https://huggingface.co/akatz-ai/MiniMax-H3-Person-Remover-LoRA/tree/main) |
| `h3_character_swap_pro4500_1000.safetensors` | `loras/` | [Akatz Character Swap — **V1 file**](https://huggingface.co/akatz-ai/MiniMax-H3-Character-Swap-LoRA/tree/main) |
| `MiniMax-H3_Ref2VA-LoRA-CrossView-Warp_v1_3500.safetensors` | `loras/` | [Cseti CrossView Warp H3](https://huggingface.co/Cseti/MiniMax-H3_Ref2VA-LoRA-CrossView-Warp_v1/tree/main) |

**Version note:** The Character Swap author currently also offers a newer LoRA. The Studio's existing code explicitly references the **older `h3_character_swap_pro4500_1000.safetensors`** file; using the new one without editing its references is not equivalent.

### E. Geometry / MoGe2 for camera-change workflows

Camera-change graphs also need MoGe-compatible inference (`LoadMoGeModel` / `MoGeInference` or equivalent compatible nodes) and a **MoGe2 ViT-L** model. Start with the [CrossViewWarp documentation](https://github.com/cseti007/ComfyUI-CrossViewWarp) and the [MoGe project](https://github.com/microsoft/MoGe). **The original ZIP does not contain the exact MoGe2 weight file or a verified version/URL**, so this is an integration-specific download, not a guaranteed one-file instruction. Check the node's model picker rather than inventing a filename.

### F. Local language models

- Install [Ollama](https://ollama.com/download), then check the [Ollama model library](https://ollama.com/search) for models available for your installation.
- Film Directions lists names such as `gemma4:12b`, `gemma4:26b-a4b-it-q4_K_M`, `qwen3.5:9b` and others. Pull the **exact available tag** your chosen backend expects, e.g. `ollama pull <valid-model-tag>`.
- **Some Retouches flows require Gemma 4 26B** (and do not silently fall back to a Qwen model). A smaller model selected in Film Directions does not automatically replace that retouch requirement.
- Two optional Qwen3.8 27B GGUF configurations use external llama-server executables and specific `.gguf` files; these are **not bundled**. Only configure them if you already understand those backends. See environment variables `H3FILM_KIWI_ROOT`, `H3FILM_LLAMA_SETTINGS` and `H3FILM_LLAMA_PORT` in the existing code.

**Model licenses:** Model files are hosted by their respective publishers, with their **own** licenses, access conditions and regional restrictions. Follow the license on every linked model page. This Studio does not redistribute H3, Krea, SAM or third-party LoRA weights.

---

## Extra workflows

The archive already includes:

- `workflows/H3Studio_1920_API_CLEAN.json`
- `workflows/Krea_reference.json`
- `workflows/Krea_reference_img2img.json`
- `workflows/Krea_reference_sref.json`
- `retouches_new_workflows/RAW_ENGINE_SOURCE_COPY.json`

The following **external retouch workflows are downloaded by the supplied installer rather than embedded in the original ZIP**:

| Local filename after installation | Source |
|---|---|
| `retouches_workflows/remove_person.json` | [Akatz Person Remover example JSON](https://huggingface.co/akatz-ai/MiniMax-H3-Person-Remover-LoRA/resolve/main/examples/Person-Remover-Window-Reroll-V1.json) |
| `retouches_workflows/swap_person.json` | [Akatz Character Swap V1 example JSON](https://huggingface.co/akatz-ai/MiniMax-H3-Character-Swap-LoRA/resolve/main/examples/H3%20Character%20Swap%20v1%20Ref2VA.json) |
| `retouches_workflows/move_camera.json` | [Cseti CrossView Warp H3 example JSON](https://huggingface.co/datasets/Cseti/ComfyUI-Workflows/resolve/main/minimax-h3/crossview-warp/crossview-warp-h3.json) |

From the **Studio release directory**, the existing download script is:

```powershell
.\INSTALLER-WORKFLOWS-RETOUCHES.ps1
```

You can instead download the JSON files manually and save them under `retouches_workflows/` with the local filenames shown. A further [LanPaint MiniMax H3 AV inpainting example](https://github.com/scraed/LanPaint) is referenced by the code for some masked-video paths. This example is **not** a substitute for the main Film Directions RAW/HD workflow.

---

## File structure

```text
H3studioFILM_V3_1_public/
├── LANCER-STUDIO-UNIFIE-7861.cmd       # Recommended Studio launcher
├── Start-H3Studio-UNIFIE-7861.ps1      # PowerShell launcher; original ComfyUI path
├── launch_h3_unified.py               # Preflight + single 7861 web server
├── h3film_studio_v3.py                # Film Directions
├── studio_web.py                       # Studio web API
├── studio_ui/                         # Browser interface
├── prompts/ + director_teams/         # Directors / prompts
├── retouches_*.py + studio_retouches.py
├── workflows/                         # Included H3 + Krea JSON workflows
├── retouches_new_workflows/           # Included RAW source / manifest
├── retouches_workflows/               # Downloaded retouch JSONs (not in initial ZIP)
├── custom_nodes/
│   ├── H3RawLatentPersist/
│   └── H3StudioRetouchExtras/
└── tests/                             # Structural and regression tests
```

The ZIP also includes historical launch scripts and release notes. For normal use, **prefer only `LANCER-STUDIO-UNIFIE-7861.cmd`**; do not start separate old Retouches and Film servers in parallel.

## Original network addresses and fixed configuration

| Service | Address | Notes |
|---|---|---|
| ComfyUI | `http://127.0.0.1:8188` | Must already be running. |
| H3 Film Studio | `http://127.0.0.1:7861` | One server, two sections. |
| Ollama | `http://127.0.0.1:11434` | Launcher can reuse or start a local Ollama process if installed. |
| Optional llama.cpp | `http://127.0.0.1:18080` | Only the configured optional backend. |
| ComfyUI path | `C:\AI\ComfyUI_windows_portable\ComfyUI` | **Hardcoded** in the supplied primary PowerShell launcher. |
| Embedded Python | `C:\AI\ComfyUI_windows_portable\python_embeded\python.exe` | **Hardcoded** in the launcher. |

**Using a different folder or port?** This release intentionally does not include an interactive setup wizard. Adapt the paths/URLs in the launch scripts and the corresponding Python modules; changing only the `COMFYUI_DIR` environment variable is **not sufficient** while the primary PowerShell launcher still uses fixed paths. The code is shared so you can make those changes yourself.

### Changing models or workflows

You can substitute your own **compatible** models, LoRAs and workflows, but the Studio may reference exact filenames and node inputs in both Python and JSON. Inspect at least:

- `workflows/H3Studio_1920_API_CLEAN.json` and the three Krea workflows;
- `h3_runtime_simple.py` and `h3film_studio_v3.py`;
- `retouches_engine.py`, `retouches_clean_plate.py`, `retouches_masked_core.py`, `retouches_crossview_core.py` and `retouches_roycho_complete_v63.py`.

A new model may require a different text encoder, VAE, latent format, attention backend, sampler, LoRA or workflow graph. **Don't assume that swapping a `.safetensors` filename is enough.** If you want to change the Studio, try using your favorite AI coding assistant: share your errors, installed ComfyUI `/object_info`, workflows and the relevant source files, and ask it to help you adapt the project to your setup. Keep backups of the original working files.

---

## Troubleshooting

**Studio says “ComfyUI not responding on :8188”**
: Start ComfyUI **first**, confirm `http://127.0.0.1:8188/object_info`, then start the Studio. Do not start a second ComfyUI instance on the same port.

**`ComfyUI introuvable` / missing `python_embeded\python.exe`**
: You did not extract the portable build to the original `C:\AI\ComfyUI_windows_portable\` location. Either use that location or edit `Start-H3Studio-UNIFIE-7861.ps1` to match your machine.

**`Nodes absents de ComfyUI EN MEMOIRE` or unknown node class**
: Compare the exact missing class to the [custom-node list](#custom-nodes--extensions), install it, examine ComfyUI import errors, and **restart ComfyUI when there is no render in progress**. The Studio may intentionally stop once after synchronizing node files. Remember that the V3.1 ZIP is missing a separate group of project-specific nodes until they are added to the release.

**`UNETLoader` / `CLIPLoader` / `VAELoader` says model not found**
: Check the **exact** filename and ComfyUI model folder. Confirm the active workflow selects Singularity (not the old NVFP4 metadata). Restart or refresh ComfyUI if the model list was cached. Not all quantizations are interchangeable.

**Ollama runs, but Director/Retouches fails**
: Run `ollama list`. An installed Ollama server is not the same as having the right model tag. Some retouch paths expect a Gemma 4 26B tag; they will not use Film Directions' smaller model automatically.

**Port 7861 already in use**
: Close the **old Studio server terminal**, not the ComfyUI render process. The unified launcher intentionally refuses to create a duplicate Studio server.

**Slow generation / out of VRAM**
: Video models and HD refinement use substantial VRAM and RAM. First check that your ComfyUI attention backend, extension versions, cache state and model/LoRA selection match the intended graph. The number of sampler steps may differ between RAW, Roycho/retake and HD refinement. Avoid blind CUDA/PyTorch upgrades.

**RAW audio is correct but HD/FINAL audio differs or breaks**
: The V3.1 package is shared **AS IS**. The author has observed audio and slowdown issues in certain retouch → HD / assembly scenarios; this README does **not** claim these are fixed. Keep the original RAW MP4 and audio as backups, check each intermediate file, and only replace the output after verifying audio. Avoid destroying existing renders or cache files to “fix” a workflow.

**Downloads from Hugging Face are missing, gated or no longer available**
: Open the **linked model repository page**, read its instructions and license, and pick the file matching the exact name used by the workflow. Some publishers require account acceptance or region-specific authorization. The project maintainer does not host the third-party weights.

---

## AS IS, community use and credits

**This is a personal project shared “AS IS”, without warranty of any kind, without guaranteed compatibility or technical support.** The author is sharing the Studio, prompts, workflows, local scripts and redistributable code as generously as possible; installing third-party software and weights is the user's responsibility. This is not a turnkey commercial product, and no particular GPU, model, operating system or output quality is guaranteed.

**Want to modify it?** Please do! Use a different LLM, model, LoRA or ComfyUI workflow, and ask your favorite AI assistant to help adapt the code and diagnose errors. Share fixes or improvements back with the community if you like. The repository is provided as a **starting point**, not a promise to maintain everyone's installation.

### Third-party licenses and model access

The Studio's source code and the external components are separate things. Each upstream project and model has its own licensing terms. **Free download does not automatically grant permission to redistribute a file.** Keep the original licenses/attributions for any upstream code bundled in a release; otherwise link to its source instead. Do not bundle third-party weights by default.

**MiniMax H3 has a territory-restricted community license.** At the time of writing, the standard community license excludes use in the **EU, UK, US and South Korea** without separate authorization. Check the [official MiniMax H3 license and Q&A](https://huggingface.co/MiniMaxAI/MiniMax-H3), and use the [official license request process](https://platform.minimax.io/h3-license) if applicable. Third-party H3 finetunes/LoRAs may inherit the same restrictions. Krea, SAM, third-party datasets and other weights can have different terms. Linking to a model is **not** a claim that every visitor may legally use it.

Credits / upstream projects used or referenced include:

- [ComfyUI](https://github.com/Comfy-Org/ComfyUI), [ComfyUI-H3Studio](https://github.com/shootthesound/ComfyUI-H3Studio), [H3 Latent Upscaler](https://github.com/LBH-123-AI/Comfyui_Minimax_h3_latent_Upscaler)
- [MaskVidExperiments](https://github.com/drozbay/MaskVidExperiments), [LanPaint](https://github.com/scraed/LanPaint), [ComfyUI-CrossViewWarp](https://github.com/cseti007/ComfyUI-CrossViewWarp), [SeedVR2](https://github.com/numz/ComfyUI-SeedVR2_VideoUpscaler)
- [MiniMax H3](https://huggingface.co/MiniMaxAI/MiniMax-H3), [Singularity](https://huggingface.co/WarmBloodAban/Minimax-h3_Singularity), [Krea-2](https://huggingface.co/Comfy-Org/Krea-2), [SAM3.1](https://huggingface.co/Comfy-Org/sam3.1)
- [Akatz Person Remover](https://huggingface.co/akatz-ai/MiniMax-H3-Person-Remover-LoRA), [Akatz Character Swap](https://huggingface.co/akatz-ai/MiniMax-H3-Character-Swap-LoRA), [Cseti CrossView-Warp LoRA](https://huggingface.co/Cseti/MiniMax-H3_Ref2VA-LoRA-CrossView-Warp_v1)

**Source license:** The Studio repository must include an appropriate `LICENSE` file **chosen by the author** before it is advertised as open source under a specific license. This README does **not** select a license on the author's behalf, and that license cannot override upstream restrictions.

---


*Made for creators who like to experiment. Take it, explore it, adapt it — and have fun making films.*

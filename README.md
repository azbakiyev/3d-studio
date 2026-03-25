# 🐉 3D Studio

AI-powered pipeline: **prompt or photo → 3D model → STL → direct print on Bambu Lab**

> Built on [extella](https://extella.ai) platform — all logic lives as reusable experts.

## 🔄 Pipeline Flow

```
Input: Text Prompt OR Local Photo (HEIC/JPG/PNG)
  │
  ▼
[1] fal_nano_banana2_text_to_image   — prompt → image (Nano Banana 2)
  │   (agent shows image, user approves or regenerates)
  ▼
[2] fal_nano_banana2_image_to_image  — denoise, remove background
  ▼
[3] fal_hunyuan3d_v3                 — image → .glb (Hunyuan3D V3)
  ▼
[4] convert_glb_to_stl               — .glb → .stl (trimesh)
  ▼
[5] rescale_stl                      — scale to target height (mm)
  ▼
[6] analyze_mesh                     — geometry check (watertight, holes, normals)
  ▼
[7] repair_and_export_mesh           — fix issues if needed (STL/3MF)
  ▼
[8] analyze_printability             — printability score 0-100, slicer tips
  ▼
[9] slice_stl_orcaslicer             — STL → G-code (OrcaSlicer CLI, background)
  ▼
[10] bambu_upload_file               — upload G-code via FTPS
  ▼
[11] bambu_control_print             — start/pause/stop via MQTT
  ▼
[12] bambu_printer_status            — monitor: progress, temp, layer
```

## 📦 Experts (16 total)

### 🎨 Image Generation
| Expert | Description |
|--------|-------------|
| `fal_nano_banana2_text_to_image` | Text → image (Nano Banana 2 / Gemini 3.1) |
| `fal_nano_banana2_image_to_image` | Image editing, denoising (up to 14 refs) |
| `fal_image_generation` | Universal image gen (FLUX, Imagen, etc.) |
| `fal_video_generation` | Text/image → video (MiniMax, WAN, Hunyuan) |

### 🧊 3D Pipeline
| Expert | Description |
|--------|-------------|
| `generate_3d_pipeline` | Orchestrator: prompt/photo → STL |
| `fal_hunyuan3d_v3` | Image → GLB (Hunyuan3D V3, fal.ai) |
| `convert_glb_to_stl` | GLB → STL (trimesh, retry logic) |
| `rescale_stl` | Scale STL to target height uniformly |
| `prepare_local_image` | HEIC/JPG/PNG → upload to fal.ai CDN |

### 🔍 Mesh Analysis & Repair
| Expert | Description |
|--------|-------------|
| `analyze_mesh` | Geometry diagnostics (watertight, holes, normals, thickness) |
| `repair_and_export_mesh` | Fix normals, fill holes, thicken walls → STL/3MF |
| `analyze_printability` | Printability score 0-100, overhang %, material tips |

### 🖨️ Printing — Bambu Lab (Phase 1)
| Expert | Description |
|--------|-------------|
| `bambu_printer_status` | MQTT status: temp, progress, layer, state |
| `bambu_upload_file` | Upload G-code via FTPS (port 990) |
| `bambu_control_print` | start/pause/resume/stop via MQTT |
| `slice_stl_orcaslicer` | STL → G-code (OrcaSlicer CLI, runs in background) |

### 🖥️ UI & Utilities
| Expert | Description |
|--------|-------------|
| `start_3d_studio` | Local web UI (Flask, self-contained, localhost:7882+) |
| `http_ping` | Check if local server is alive |

## 🛠️ Setup

### Requirements
- [extella Desktop](https://extella.ai)
- [OrcaSlicer](https://github.com/SoftFever/OrcaSlicer/releases) — for slicing STL→G-code
- fal.ai API key

### KV Store Keys (stored in extella)
| Key | Description |
|-----|-------------|
| `fal_api_key` | fal.ai API key |
| `bambu_printer_ip` | Printer IP address |
| `bambu_serial_number` | Printer serial number |
| `bambu_access_code` | 8-digit LAN access code |
| `extella_api_token` | API token for nested experts |

### Bambu Lab LAN Mode Setup
1. On printer screen: **Settings → Network → LAN Mode Liveview → Enable**
2. Note: IP address + Access Code (8 digits)
3. Serial number: Bambu Studio → Device → three dots → Device Info

## 📚 Materials Library

| Material | Nozzle | Bed | Speed | Best for |
|----------|--------|-----|-------|----------|
| PLA | 190-220°C | 25-60°C | 40-80 mm/s | Prototypes, figures |
| PETG | 230-250°C | 70-85°C | 30-60 mm/s | Functional parts |
| TPU | 220-240°C | 30-60°C | 15-30 mm/s | Flexible parts |
| ABS | 230-250°C | 100-110°C | 40-70 mm/s | High-temp/mechanical |

## 🔮 Roadmap

### Phase 2: Universal Printer Support
- [ ] OctoPrint adapter (covers Creality, older Prusa)
- [ ] Moonraker/Klipper adapter (Voron, RatRig)
- [ ] PrusaLink adapter
- [ ] `printer_manager` orchestrator (auto-detect printer type)

### Phase 3: Smart Features
- [ ] Camera-based defect detection (needs webcam)
- [ ] Print queue management
- [ ] Automatic material profile selection
- [ ] Multi-material support

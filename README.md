# 🐉 3D Studio

AI-powered pipeline: **prompt or photo → 3D model → STL → direct print on Bambu Lab**

> Built on [extella](https://extella.ai) platform — all logic lives as reusable experts.

## 🔄 Full Pipeline Flow

```
Input: Text Prompt OR Local Photo (HEIC/JPG/PNG)
  │
  ▼
[1] fal_nano_banana2_text_to_image   — prompt → image (Nano Banana 2 / Gemini 3.1)
  │   (agent shows image, user approves or regenerates)
  ▼
[2] fal_nano_banana2_image_to_image  — denoise, remove background for 3D
  ▼
[3] fal_hunyuan3d_v3                 — image → .glb (Hunyuan3D V3)
  ▼
[4] convert_glb_to_stl               — .glb → .stl (trimesh, retry on CDN 503)
  ▼
[5] rescale_stl                      — scale to target height (mm), uniform
  ▼
[6] analyze_mesh                     — geometry check (watertight, holes, normals)
  ▼
[7] repair_and_export_mesh           — fix issues if needed (STL/3MF)
  ▼
[8] analyze_printability             — printability score 0-100, material tips
  ▼
[9] slice_stl_orcaslicer             — STL → G-code (OrcaSlicer CLI, background, no UI)
  ▼
[10] bambu_upload_file               — upload G-code via FTPS (port 990)
  ▼
[11] bambu_control_print             — start/pause/stop via MQTT
  ▼
[12] bambu_printer_status            — monitor: progress %, temp, layer, ETA
  ▼
[13] bambu_camera_snapshot           — capture live snapshot from printer camera
```

## 📦 Experts (21 total)

### 🎨 Image Generation
| Expert | Description |
|--------|-------------|
| `fal_nano_banana2_text_to_image` | Text → image (Nano Banana 2 / Gemini 3.1 Flash Image) |
| `fal_nano_banana2_image_to_image` | Image editing, denoising, background removal (up to 14 refs) |
| `fal_image_generation` | Universal image gen (FLUX Pro, Imagen 4, etc.) |
| `fal_video_generation` | Text/image → video (MiniMax, WAN, Hunyuan Video) |

### 🧊 3D Pipeline
| Expert | Description |
|--------|-------------|
| `generate_3d_pipeline` | Orchestrator: prompt/photo → STL (calls sub-experts via API) |
| `fal_hunyuan3d_v3` | Image/text → GLB (Hunyuan3D V3, fal.ai, PBR textures) |
| `convert_glb_to_stl` | GLB → STL (trimesh, 5-retry logic for fal.ai CDN 503) |
| `rescale_stl` | Scale STL to exact target height (uniform, preserves proportions) |
| `prepare_local_image` | HEIC/JPG/PNG → convert + upload to fal.ai CDN |

### 🔍 Mesh Analysis & Repair
| Expert | Description |
|--------|-------------|
| `analyze_mesh` | Geometry diagnostics: watertight, holes, normals, thickness |
| `repair_and_export_mesh` | Smart repair: fix normals / fill holes / thicken walls → STL or 3MF |
| `analyze_printability` | Printability score 0-100: overhangs, thin walls, bed adhesion, material tips |

### 🖨️ Printing — Bambu Lab (Phase 1)
| Expert | Description |
|--------|-------------|
| `bambu_printer_status` | Real-time status via MQTT: temp, progress %, layer, ETA, state |
| `bambu_upload_file` | Upload G-code to printer via FTPS (port 990, LAN) |
| `bambu_control_print` | Print control via MQTT: start / pause / resume / stop |
| `slice_stl_orcaslicer` | STL → G-code using OrcaSlicer CLI (runs in background, no UI shown) |
| `bambu_camera_snapshot` | Capture live JPEG snapshot from printer camera (port 6000, bambulab lib) |

### 🛠️ Utilities & Vision
| Expert | Description |
|--------|-------------|
| `start_3d_studio` | Local web UI (Flask, self-contained, localhost, dark theme) |
| `http_ping` | Check if any local or remote server is alive |
| `scan_ports` | Scan TCP ports on any host (used for printer diagnostics) |
| `read_image_text` | Read/analyze images using Gemini 2.0 Flash vision model (fal.ai) |

## 🛠️ Setup

### Requirements
- [extella Desktop](https://extella.ai)
- [OrcaSlicer](https://github.com/SoftFever/OrcaSlicer/releases) — for slicing STL → G-code
- [ffmpeg](https://ffmpeg.org) — `brew install ffmpeg` (for video utilities)
- fal.ai API key

### KV Store Keys (stored in extella)
| Key | Description |
|-----|-------------|
| `fal_api_key` | fal.ai API key |
| `bambu_printer_ip` | Printer IP address (e.g. 192.168.8.47) |
| `bambu_serial_number` | Printer serial number |
| `bambu_access_code` | 8-digit LAN access code |
| `extella_api_token` | API token for nested experts (auto-generated) |

### Bambu Lab LAN Mode Setup
1. On printer screen: **Settings → Network → LAN Only Mode → Enable**
2. Note: IP address + Access Code (8 digits)
3. Serial number: Bambu Studio → Device → three dots → Device Info
4. Camera: **Settings → Camera Options → Video → On**

### Camera Access (Bambu A1)
- Uses **port 6000** (not 322 — that's X1C only)
- Protocol: TCP with `bambu-lab-cloud-api` library (`JPEGFrameStream`)
- Works in **idle mode** — no active print required!
- Requires `Video: On` in Camera Options

## 📚 Materials Library

| Material | Nozzle | Bed | Speed | Best for |
|----------|--------|-----|-------|----------|
| PLA | 190-220°C | 25-60°C | 40-80 mm/s | Prototypes, figures |
| PETG | 230-250°C | 70-85°C | 30-60 mm/s | Functional parts |
| TPU | 220-240°C | 30-60°C | 15-30 mm/s | Flexible parts |
| ABS | 230-250°C | 100-110°C | 40-70 mm/s | High-temp/mechanical |

## 🔮 Roadmap

### Phase 2: Smart Print Monitoring
- [ ] AI defect detection: camera frame → Nano Banana 2 → pause on defect
- [ ] Timelapse: auto-capture frames → MP4 video
- [ ] Print queue management
- [ ] Telegram notifications on print complete

### Phase 3: Universal Printer Support
- [ ] OctoPrint adapter (Creality, older Prusa)
- [ ] Moonraker/Klipper adapter (Voron, RatRig)
- [ ] PrusaLink adapter
- [ ] `printer_manager` orchestrator (auto-detect printer type)

### Phase 4: Web App / SaaS
- [ ] Deploy to Railway/Render
- [ ] User auth + personal fal.ai keys
- [ ] Model gallery and history
- [ ] Freemium monetization

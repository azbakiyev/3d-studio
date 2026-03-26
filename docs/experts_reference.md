# Experts Reference — 3D Studio

All 21 experts stored in extella platform. Parameters and usage below.

---

## 🎨 Image Generation

### `fal_nano_banana2_text_to_image`
Text → image via Nano Banana 2 (Gemini 3.1 Flash Image) on fal.ai.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `prompt` | `""` | Text description |
| `num_images` | `1` | Number of images (1-4) |
| `aspect_ratio` | `"1:1"` | `1:1`, `16:9`, `9:16`, `4:3`, `auto` |
| `resolution` | `"1K"` | `0.5K`, `1K`, `2K`, `4K` |
| `output_format` | `"png"` | `png` / `jpeg` |
| `thinking` | `""` | `minimal` or `high` (+$0.002) |
| `enable_web_search` | `0` | `1` = web search (+$0.015) |
| `fal_api_key_value` | `""` | Direct API key (or from KV Store) |

**Cost:** $0.08/image. 2K = 1.5x, 4K = 2x price.

---

### `fal_nano_banana2_image_to_image`
Context-aware image editing — no masks. Up to 14 reference images.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `prompt` | `""` | Editing instructions |
| `image_urls` | `""` | URLs comma-separated (up to 14) |
| `num_images` | `1` | Number of output variants |
| `aspect_ratio` | `"auto"` | Output aspect ratio |
| `resolution` | `"1K"` | `1K` / `2K` / `4K` |
| `thinking` | `""` | `minimal` or `high` |
| `fal_api_key_value` | `""` | Direct API key |

---

### `fal_image_generation`
Universal image generation via any fal.ai model.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `prompt` | `""` | Text prompt |
| `model_id` | `"fal-ai/flux-pro/v1.1-ultra"` | Any fal.ai image model |
| `image_size` | `"landscape_16_9"` | `square_hd`, `portrait_4_3`, etc. |
| `num_images` | `1` | Number of images |
| `seed` | `-1` | Seed (-1 = random) |
| `negative_prompt` | `""` | Negative prompt |
| `fal_api_key_value` | `""` | Direct API key |

---

### `fal_video_generation`
Text-to-video and image-to-video via fal.ai.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `prompt` | `""` | Text prompt |
| `image_url` | `""` | Input image URL (for image-to-video) |
| `model_id` | `"fal-ai/minimax/video-01"` | Video model ID |
| `duration` | `6` | Duration in seconds |
| `resolution` | `"720p"` | Video resolution |
| `aspect_ratio` | `"16:9"` | Aspect ratio |
| `fal_api_key_value` | `""` | Direct API key |

---

## 🧊 3D Pipeline

### `generate_3d_pipeline` (orchestrator)
Two-mode nested expert. Calls sub-experts via extella REST API.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `mode` | `"generate_image"` | `generate_image` or `image_to_3d` |
| `prompt` | `""` | For generate_image mode |
| `source_image_url` | `""` | For image_to_3d mode |
| `generate_type` | `"Normal"` | `Normal`, `LowPoly`, `Geometry` |
| `face_count` | `300000` | Polygon count (40K–1.5M) |
| `stl_format` | `"binary"` | `binary` or `ascii` |

---

### `fal_hunyuan3d_v3`
Generates 3D model (.glb + preview) from image or text.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `mode` | `"auto"` | `auto`, `image`, `text` |
| `input_image_url` | `""` | Image URL for image-to-3D |
| `prompt` | `""` | Text for text-to-3D |
| `generate_type` | `"Normal"` | `Normal` (textured), `LowPoly`, `Geometry` |
| `face_count` | `500000` | Polygons: 40K–1.5M |
| `enable_pbr` | `1` | PBR materials (metal, roughness) |
| `fal_api_key_value` | `""` | Direct API key |

**Output:** `.glb` + preview `.png` → `~/Downloads/`

---

### `convert_glb_to_stl`
Converts GLB to STL using trimesh. Retry logic for fal.ai CDN 503 errors.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `input_url` | `""` | Remote GLB URL |
| `input_path` | `""` | Local GLB path |
| `output_path` | `""` | Output STL path (auto if empty) |
| `stl_format` | `"binary"` | `binary` or `ascii` |
| `merge_meshes` | `1` | Merge all scene meshes into one |
| `max_retries` | `5` | Retry attempts on 503 |
| `retry_delay` | `15` | Seconds between retries |

---

### `rescale_stl`
Uniformly scales STL to exact target height preserving proportions.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `input_path` | `""` | Input STL path |
| `target_height_mm` | `50` | Target height in mm |
| `scale_axis` | `"y"` | Which axis is "height": `x`, `y`, `z` |
| `output_path` | `""` | Output path (auto if empty) |

**Note:** Uses `y` axis by default (tallest dimension). Auto-detects max axis.

---

### `prepare_local_image`
Converts local image (HEIC/JPG/PNG) and uploads to fal.ai CDN for processing.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `input_path` | `""` | Local file path |
| `output_format` | `"png"` | `png` or `jpeg` |
| `fal_api_key_value` | `""` | Direct API key |

**Supports:** HEIC (iPhone), JPG, PNG, WebP, BMP

---

## 🔍 Mesh Analysis & Repair

### `analyze_mesh`
Diagnoses 3D mesh geometry for print-readiness.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `input_path` | `""` | STL/GLB local path |
| `input_url` | `""` | Remote URL |
| `min_thickness_mm` | `0.8` | Min wall thickness threshold |
| `print_size_mm` | `50` | Scale context for analysis |

**Returns:** `needs_repair`, `needs_fill_holes`, `needs_thickening`, `severity`, `issues[]`, `recommendations[]`

---

### `repair_and_export_mesh`
Applies only needed repairs based on `analyze_mesh` output flags.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `input_path` | `""` | STL/GLB path |
| `do_repair` | `0` | Fix normals + remove degenerate faces |
| `do_fill_holes` | `0` | Close open boundary edges |
| `do_thicken` | `0` | Offset vertices along normals |
| `thicken_mm` | `1.0` | Thickness offset in mm |
| `output_format` | `"stl"` | `stl` or `3mf` |
| `output_path` | `""` | Output path (auto if empty) |

---

### `analyze_printability`
Scores model printability for FDM 3D printing (0-100).

| Parameter | Default | Description |
|-----------|---------|-------------|
| `input_path` | `""` | STL/GLB path |
| `material` | `"PLA"` | `PLA`, `PETG`, `TPU`, `ABS` |
| `print_size_mm` | `50` | Target print height in mm |
| `overhang_angle_deg` | `45` | Overhang threshold |
| `min_wall_mm` | `0.8` | Min wall thickness |

**Score:** 85-100 = Ready ✅, 60-84 = Warning ⚠️, 40-59 = Fix needed 🔶, <40 = Not ready ❌

**Returns:** `score`, `verdict`, `needs_supports`, `brim_recommended`, `issues[]`, `slicer_hints[]`, `material_tips[]`

---

## 🖨️ Printing — Bambu Lab

### `bambu_printer_status`
Connects via MQTT (port 8883, TLS) and gets real-time printer state.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `printer_ip` | `""` | Printer IP (or from KV: `bambu_printer_ip`) |
| `serial_number` | `""` | Serial (or from KV: `bambu_serial_number`) |
| `access_code` | `""` | 8-digit code (or from KV: `bambu_access_code`) |
| `ping_only` | `0` | `1` = port scan only, skip MQTT |
| `timeout_sec` | `20` | MQTT connection timeout |

**Returns:** `state`, `progress_pct`, `current_layer`, `total_layers`, `nozzle_temp`, `bed_temp`, `remaining_min`, `print_file`

---

### `bambu_upload_file`
Uploads G-code to printer internal storage via FTPS (port 990).

| Parameter | Default | Description |
|-----------|---------|-------------|
| `file_path` | `""` | Local .gcode file path |
| `printer_ip` | `""` | Printer IP |
| `access_code` | `""` | LAN access code (password for FTPS) |
| `remote_filename` | `""` | Filename on printer (auto if empty) |

**Login:** `bblp` / `<access_code>` via FTPS to `/cache/`

---

### `bambu_control_print`
Controls print via MQTT commands.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `command` | `"status"` | `status`, `start`, `pause`, `resume`, `stop` |
| `filename` | `""` | File to print (for `start` command) |
| `printer_ip` | `""` | Printer IP |
| `serial_number` | `""` | Serial number |
| `access_code` | `""` | LAN access code |

---

### `slice_stl_orcaslicer`
Slices STL to G-code using OrcaSlicer CLI. Runs in background — no UI shown.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `input_stl` | `""` | Input STL file path |
| `output_dir` | `"/tmp"` | Output directory for .gcode |
| `printer_profile` | `"Bambu Lab A1"` | Printer profile in OrcaSlicer |
| `filament_profile` | `"Bambu PLA Basic"` | Filament profile |
| `layer_height` | `0.2` | Layer height in mm |
| `infill_density` | `15` | Infill percentage |
| `support_type` | `"normal"` | `none`, `normal`, `tree` |
| `orca_path` | `"/Applications/OrcaSlicer.app/Contents/MacOS/OrcaSlicer"` | Binary path |

---

### `bambu_camera_snapshot`
Captures live JPEG snapshot from Bambu A1 printer camera.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `printer_ip` | `""` | Printer IP |
| `access_code` | `""` | LAN access code |
| `output_path` | `"~/Downloads/bambu_snapshot.jpg"` | Save path |
| `timeout_sec` | `10` | Connection timeout |

**Protocol:** TCP on port 6000 using `bambu-lab-cloud-api` (`JPEGFrameStream`)
**Works in idle mode** — no active print required!
**Requires:** `Video: On` in printer Camera Options settings

---

## 🛠️ Utilities & Vision

### `start_3d_studio`
Launches local web UI for the full pipeline. Opens browser automatically.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `port` | `7882` | Local port (auto-increments on restart) |
| `fal_api_key_value` | `""` | fal.ai API key |
| `api_token_value` | `""` | extella API token for sub-expert calls |

**Architecture:** Flask → extella API (localhost:5000) → sub-experts → fal.ai

---

### `http_ping`
Checks if a URL is reachable. Returns status code, response time, body snippet.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `url` | `""` | URL to check |
| `timeout` | `5` | Timeout in seconds |

---

### `scan_ports`
Scans TCP ports on any host. Used for printer and server diagnostics.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `host` | `""` | Hostname or IP |
| `ports` | `"80,443,8080,8883"` | Comma-separated port list |
| `timeout` | `2` | Per-port timeout in seconds |

**Bambu A1 open ports:** 990 (FTPS), 6000 (Camera), 8883 (MQTT)

---

### `read_image_text`
Analyzes images using Gemini 2.0 Flash vision via fal.ai LLM API.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `image_urls` | `""` | Image URLs comma-separated (up to 4) |
| `question` | `""` | Question or instruction for the model |
| `model` | `"google/gemini-2.0-flash-001"` | Vision model ID |
| `fal_api_key_value` | `""` | Direct API key |

**Use cases:** Read text from screenshots, analyze images, describe content

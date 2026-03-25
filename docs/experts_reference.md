# Experts Reference

All experts are stored in extella platform. This document describes parameters and usage.

---

## 🎨 Image Generation

### `fal_nano_banana2_text_to_image`
Generates images from text using Nano Banana 2 (Gemini 3.1 Flash Image) on fal.ai.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `prompt` | `""` | Text description |
| `num_images` | `1` | Number of images |
| `aspect_ratio` | `"1:1"` | `1:1`, `16:9`, `9:16`, `4:3` |
| `resolution` | `"1K"` | `0.5K`, `1K`, `2K`, `4K` |
| `output_format` | `"png"` | `png` / `jpeg` |
| `thinking` | `""` | `minimal` or `high` (+$0.002) |
| `fal_api_key_value` | `""` | Direct API key (or from KV Store) |

**Cost:** $0.08/image

---

### `fal_nano_banana2_image_to_image`
Context-aware image editing — no masks needed. Up to 14 reference images.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `prompt` | `""` | Editing instructions |
| `image_urls` | `""` | URLs comma-separated (up to 14) |
| `num_images` | `1` | Number of variants |
| `aspect_ratio` | `"auto"` | Aspect ratio |
| `resolution` | `"1K"` | `1K` / `2K` / `4K` |
| `fal_api_key_value` | `""` | Direct API key |

---

## 🧊 3D Pipeline

### `generate_3d_pipeline` (orchestrator)
Two-mode orchestrator. Calls sub-experts via REST API.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `mode` | `"generate_image"` | `generate_image` or `image_to_3d` |
| `prompt` | `""` | For generate_image mode |
| `source_image_url` | `""` | For image_to_3d mode |
| `generate_type` | `"Normal"` | `Normal`, `LowPoly`, `Geometry` |
| `face_count` | `300000` | Polygon count |
| `stl_format` | `"binary"` | `binary` or `ascii` |

---

### `fal_hunyuan3d_v3`
Generates 3D model (.glb) from image or text using Hunyuan3D V3.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `mode` | `"auto"` | `auto`, `image`, `text` |
| `input_image_url` | `""` | Image URL for image-to-3D |
| `prompt` | `""` | Text for text-to-3D |
| `generate_type` | `"Normal"` | `Normal`, `LowPoly`, `Geometry` |
| `face_count` | `500000` | 40K–1.5M polygons |
| `enable_pbr` | `1` | PBR materials |
| `fal_api_key_value` | `""` | Direct API key |

---

### `convert_glb_to_stl`
Converts GLB to STL using trimesh. Retry logic for fal.ai CDN 503 errors.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `input_url` | `""` | Remote GLB URL |
| `input_path` | `""` | Local GLB path |
| `output_path` | `""` | Output STL path (auto if empty) |
| `stl_format` | `"binary"` | `binary` or `ascii` |
| `merge_meshes` | `1` | Merge all meshes into one |
| `max_retries` | `5` | Retry attempts on 503 |
| `retry_delay` | `15` | Seconds between retries |

---

### `rescale_stl`
Uniformly scales STL to target height.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `input_path` | `""` | Input STL path |
| `target_height_mm` | `50` | Target height in mm |
| `scale_axis` | `"y"` | Axis to use as height: `x`, `y`, `z` |
| `output_path` | `""` | Output path (auto if empty) |

---

### `prepare_local_image`
Converts local image (HEIC/JPG/PNG) and uploads to fal.ai CDN.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `input_path` | `""` | Local file path |
| `output_format` | `"png"` | `png` or `jpeg` |
| `fal_api_key_value` | `""` | Direct API key |

---

## 🔍 Mesh Analysis & Repair

### `analyze_mesh`
Diagnoses 3D mesh geometry issues.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `input_path` | `""` | STL/GLB local path |
| `input_url` | `""` | Remote URL |
| `min_thickness_mm` | `0.8` | Min wall thickness threshold |
| `print_size_mm` | `50` | Scale context for analysis |

**Returns:** `needs_repair`, `needs_fill_holes`, `needs_thickening`, `issues[]`, `recommendations[]`

---

### `repair_and_export_mesh`
Applies only needed repairs based on analyze_mesh output.

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
Scores 3D model printability for FDM printing.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `input_path` | `""` | STL/GLB path |
| `material` | `"PLA"` | `PLA`, `PETG`, `TPU`, `ABS` |
| `print_size_mm` | `50` | Target print height in mm |
| `overhang_angle_deg` | `45` | Overhang threshold |
| `min_wall_mm` | `0.8` | Min wall thickness |

**Returns:** `score` (0-100), `verdict`, `needs_supports`, `brim_recommended`, `issues[]`, `slicer_hints[]`, `material_tips[]`

**Score legend:** 85-100 = Ready ✅, 60-84 = Warning ⚠️, 40-59 = Fix needed 🔶, <40 = Not ready ❌

---

## 🖨️ Printing — Bambu Lab

### `bambu_printer_status`
Connects via MQTT and retrieves real-time printer state.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `printer_ip` | `""` | Printer IP (or from KV: `bambu_printer_ip`) |
| `serial_number` | `""` | Serial number (or from KV: `bambu_serial_number`) |
| `access_code` | `""` | 8-digit LAN code (or from KV: `bambu_access_code`) |
| `ping_only` | `0` | `1` = check ports only, no MQTT |
| `timeout_sec` | `20` | MQTT connection timeout |

**Returns:** `state`, `progress_pct`, `current_layer`, `total_layers`, `nozzle_temp`, `bed_temp`, `remaining_min`, `print_file`

---

### `bambu_upload_file`
Uploads G-code to printer via FTPS (port 990).

| Parameter | Default | Description |
|-----------|---------|-------------|
| `file_path` | `""` | Local .gcode file path |
| `printer_ip` | `""` | Printer IP |
| `access_code` | `""` | LAN access code |
| `remote_filename` | `""` | Filename on printer (auto if empty) |

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
Slices STL to G-code using OrcaSlicer CLI (runs in background, no UI).

| Parameter | Default | Description |
|-----------|---------|-------------|
| `input_stl` | `""` | Input STL path |
| `output_dir` | `"/tmp"` | Output directory |
| `printer_profile` | `"Bambu Lab A1"` | Printer name in OrcaSlicer |
| `filament_profile` | `"Bambu PLA Basic"` | Filament profile |
| `layer_height` | `0.2` | Layer height in mm |
| `infill_density` | `15` | Infill % |
| `support_type` | `"normal"` | `none`, `normal`, `tree` |
| `orca_path` | `"/Applications/OrcaSlicer.app/Contents/MacOS/OrcaSlicer"` | OrcaSlicer binary |

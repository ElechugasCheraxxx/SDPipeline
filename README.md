# SDPipeline

**Native macOS front-end for Stable Diffusion — SwiftUI + Metal, built for Apple Silicon.**

[![Swift](https://img.shields.io/badge/Swift-5.9%2B-F05038.svg?)](#)
[![macOS](https://img.shields.io/badge/macOS-14%2B%20Sonoma-000000.svg?)](#)
[![SwiftUI](https://img.shields.io/badge/SwiftUI-lifecycle-0A84FF.svg?)](#)
[![License](https://img.shields.io/badge/license-GPL--3.0-blue.svg)](LICENSE)

> A desktop control surface for a local Stable Diffusion backend: 53k lines of Swift across
> 105 files, with the GPU pipeline, asset library and export path handled natively on Apple Silicon.

---

## Why native

Running SD from the browser means fighting the tab: no filesystem access, every image crosses
a base64 boundary, and the browser tab is the single point of failure for a 20-minute render.

SDPipeline is a native app that owns the whole path — it talks to the local backend over HTTP,
manages the asset library on disk, and keeps render state alive across app launches.

---

## Architecture

```
┌─────────────────────────────────────────────┐
│  SwiftUI layer                               │
│  ContentView · Dashboard · JobQueue · Gallery│
├─────────────────────────────────────────────┤
│  Engines (28) — one per pipeline stage       │
│  ControlNet · Img2Img · Inpainting · ADetailer│
│  IC-Light · HiRes Finish · Tiled Upscaler    │
│  ControlNet · IP-Adapter · LoRA · Scene      │
├─────────────────────────────────────────────┤
│  Managers (20) — lifecycle & policy          │
│  ModelManager · JobQueue · Batch · Backup    │
│  Docker · CloudScheduler · Integrity         │
├─────────────────────────────────────────────┤
│  Stores (4) — persistence                    │
│  AssetStore · PromptVersioning · Tagging     │
│  AssetVersioning                             │
├─────────────────────────────────────────────┤
│  Data plane                                  │
│  CoreDataPersistence · PromptDatabase        │
│  Data_Crypto · VaultCryptoEngine             │
└─────────────────────────────────────────────┘
```

### Selected modules

| Module | Lines | What it does |
|---|---|---|
| `ControlNetEngine.swift` | 1.227 | ControlNet preprocessor/adapter graph |
| `Img2ImgEngine.swift` | 1.026 | Image-to-image with strength scheduling |
| `InpaintingEngine.swift` | 922 | Masked region regeneration, feather + blend |
| `CharacterEngine.swift` | 893 | Character LoRA composition across regions |
| `BackupManager.swift` | 828 | Scheduled + integrity-verified backups |
| `BatchEngine.swift` | 809 | Queue-driven batch generation |
| `SceneEngine.swift` | 800 | Multi-layer scene composition |
| `PromptDatabase.swift` | 1.051 | Prompt corpus with versioning and search |
| `SetupAssistantView.swift` | 996 | First-run backend detection and wiring |

---

## Notable capabilities

**GPU & performance**
- `MpsOptimizer.swift` — Metal Performance Shaders tuning for unified memory
- `GPUMonitor.swift` — live VRAM/inference telemetry
- `DepthCacheEngine` / `XYPlotEngine` / `HiResFinishEngine` — tiled upscaling with cache reuse

**Reliability**
- `IntegrityManager` / `AppHardeningManager` — runtime integrity checks
- `BackupManager` + `ZeroKnowledgeLog` — verifiable backup chain, tamper-evident log
- `JobQueueManager` — survives app restart; no render lost on quit
- `SandboxManager` — filesystem boundary enforcement

**Asset management**
- `AssetStore` + `AssetVersioningStore` — full version history per asset
- `TaggingEngine`, `ClipSearchEngine` — CLIP-embedding semantic search
- `EmbeddingsManager` — embedding-based prompt injection and retrieval
- `LoRAManager` / `LoRAEncapsulationEngine` / `KohyaTrainingManager` — LoRA lifecycle

**Export & interop**
- `IPTCMetadataWriter` — writes generation metadata into image headers
- `SteganographyEngine` — payload embedding in LSB channel
- `PipelineConnector` / `DAMBridge` / `SDService` — backend adapters
- `PostProductionEngine`, `ACEScgColorEngine`, `VisionAestheticsEngine` — ACEScg color pipeline

**12 engines for specific models:** ControlNet, IP-Adapter, IC-Light, ADetailer, HiRes-Fix,
Tiled Upscaler, Wildcard, Cinematic Filter, Scene, Character, Img2Img, Inpainting.

---

## Build

Requires macOS 14+ and Xcode 15+.

```bash
# Open in Xcode
open SDPipeline.xcodeproj

# Or build from CLI
xcodebuild -project SDPipeline.xcodeproj \
           -scheme SDPipeline \
           -configuration Release \
           build
```

Then point the app at your local Stable Diffusion backend (default `http://127.0.0.1:7860`)
in Settings → Backend.

---

## Project layout

```
SDPipeline/
├── ContentView*.swift          app shell
├── SetupAssistantView.swift    first-run configuration
├── DashboardView.swift         throughput / VRAM telemetry
├── GalleryView.swift           asset browser
├── JobQueueView.swift          queue + progress
├── PublishView.swift           export targets
├── *Engine.swift               (28) pipeline stages
├── *Manager.swift              (20) lifecycle & policy
├── *Store.swift                (4)  persistence
├── *View.swift                 (18) SwiftUI surfaces
└── Models*.swift               data layer + Core Data schema
```

---

## Status

Active development. Interfaces evolve as backend APIs change.

## License

GPL-3.0 — see [`LICENSE`](LICENSE).


---

<div align="center">

**José Miguel Tirado Quiroz** · Security & Infrastructure Engineer · Lima, Perú

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://linkedin.com/in/jose-miguel-tirado-quiroz)
[![GitHub](https://img.shields.io/badge/GitHub-ElechugasCheraxxx-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/ElechugasCheraxxx)

</div>

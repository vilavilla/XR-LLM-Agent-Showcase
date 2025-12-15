# Architecture Diagram - Visual Design Specification

## For: XR-LLM-Agent-Showcase README.md

---

## 📐 DIAGRAM 1: System Architecture Overview (Main Diagram)

### Layout Type
**Horizontal Flow Diagram** with 4 main vertical swimlanes

### Dimensions
- **Canvas Size**: 1200px width × 800px height
- **Style**: Modern tech diagram with rounded corners, drop shadows, and color-coded sections

---

### SWIMLANE 1: User Layer (Left Side)
**Color**: Light Blue (#E3F2FD)

**Components**:
1. **User Icon** (person silhouette)
   - Position: Top-left
   - Label: "VR User"
   - Sub-elements:
     - Microphone icon (input)
     - Headset icon (output)

2. **Interaction Box**
   - Text: "Voice Input"
   - Arrow pointing right →
   - Label on arrow: "Raw Audio (44.1kHz PCM)"

---

### SWIMLANE 2: Unity Frontend (C#)
**Color**: Purple (#E1BEE7)

**Components**:
1. **Wake Word Detector**
   - Shape: Rounded rectangle
   - Icon: 🎤
   - Text: "Windows Speech Recognition"
   - Label: "Keyword: 'Hey Marc'"
   - Arrow down ↓

2. **Audio Processor**
   - Shape: Hexagon
   - Text: "WhisperManager"
   - Technical details (smaller font):
     - "Downsampling: 44.1 → 16kHz"
     - "Circular Buffer"
     - "VAD Segmentation"
   - Arrow right → (crosses to Python swimlane)
   - Label on arrow: "WebSocket (ws://127.0.0.1:8000)"

3. **Avatar Controller**
   - Shape: Rounded rectangle
   - Icon: 🤖
   - Text: "UnifiedRobotController"
   - Sub-components (nested boxes):
     - "Animator (Blend Shapes)"
     - "LED System (State Machine)"
     - "HeadLipsyncByRMS"
   - Input arrow from TTS ←
   - Output arrow to User →

4. **UI System**
   - Shape: Small rectangle
   - Text: "UIChatController"
   - Connected to Avatar Controller with dotted line

---

### SWIMLANE 3: Python Backend
**Color**: Green (#C8E6C9)

**Top Section: FastAPI Server**
1. **API Gateway**
   - Shape: Rounded rectangle
   - Text: "FastAPI (uvicorn)"
   - Port label: ":8000"
   - Endpoints listed:
     - `/generate`
     - `/chat`
     - `/rag/ask`
   - Arrow down ↓

**Middle Section: AI Pipeline**
2. **Speech-to-Text Module**
   - Shape: Hexagon
   - Icon: 🎧
   - Text: "Whisper.cpp"
   - Technical details:
     - "Model: ggml-base.bin (140MB)"
     - "Load time: ~67ms"
     - "Languages: es/en/ca"
   - Arrow down ↓
   - Output label: "Transcribed Text"

3. **Processing Hub** (central node)
   - Shape: Circle (decision point)
   - Text: "Router"
   - Three arrows branching out:
     - Arrow to LLM →
     - Arrow to RAG System →
     - Arrow to Clinical Context →

4. **LLM Interface**
   - Shape: Rounded rectangle
   - Text: "Ollama Client"
   - Arrow right → (to Ollama swimlane)

5. **RAG System**
   - Shape: Rounded rectangle with database icon
   - Text: "RAG V2 Engine"
   - Sub-components:
     - "ChromaDB (Vector Store)"
     - "Sentence Transformers"
     - "PDF Indexer"
   - Connected to LLM Interface with bidirectional arrow ↔

6. **Text-to-Speech Module**
   - Shape: Hexagon
   - Icon: 🔊
   - Text: "PiperTTS Controller"
   - Technical details:
     - "Voice: es_ES-male-medium"
     - "Latency: ~143ms/24 words"
   - Arrow left ← (back to Unity)
   - Output label: "Audio Stream (WAV)"

---

### SWIMLANE 4: LLM Runtime
**Color**: Orange (#FFE0B2)

**Components**:
1. **Ollama Server**
   - Shape: Large rounded rectangle
   - Text: "Ollama Runtime"
   - Icon: 🦙
   - Sub-sections (stacked):
     - **Model Box 1**:
       - "Mistral 7B Instruct v0.3"
       - "Quantization: Q4_K_M (4-bit)"
       - "Size: ~4.1GB"
     - **Model Box 2**:
       - "Llama 3 8B Instruct"
       - "Quantization: Q4_K_M"
       - "Size: ~4.7GB"
   - Technical specs (bottom):
     - "Context Window: 4096 tokens"
     - "GPU Layers: 24/32"
     - "Speed: ~30 tok/s"
   - Arrow left ← (back to Python Backend)
   - Output label: "Generated Response"

2. **Resource Monitor** (bottom of swimlane)
   - Shape: Small status box
   - Text: "VRAM Usage: ~5.2GB"
   - Bar chart icon

---

### CROSS-CUTTING ELEMENTS

**1. Latency Timeline (Top)**
- Horizontal timeline across all swimlanes
- Markers at key stages:
  - T+0ms: Wake word detected
  - T+67ms: Whisper loaded
  - T+500ms: Transcription complete
  - T+800ms: LLM response generated
  - T+943ms: Audio synthesized
  - T+1000ms: Avatar starts speaking
- Color gradient from red → yellow → green

**2. Data Flow Arrows**
- Main path: Thick solid arrows (blue)
- Alternative paths: Dashed arrows (gray)
- Error paths: Red dotted arrows
- Arrow labels with data format (e.g., "JSON", "WAV", "Text")

**3. Technology Badges (Bottom)**
- Small icon badges showing:
  - Unity logo
  - Python logo
  - WebSocket icon
  - GPU icon (CUDA)

---

### COLOR CODING LEGEND (Bottom-Right Corner)
```
┌─────────────────────────────────┐
│ LEGEND                          │
├─────────────────────────────────┤
│ 🔵 User Interaction             │
│ 🟣 Unity (C# .NET)              │
│ 🟢 Python Backend               │
│ 🟠 AI Inference (Ollama)        │
│ ──▶ Synchronous Call            │
│ ··▶ Asynchronous Event          │
│ ⚡ Performance Critical Path    │
└─────────────────────────────────┘
```

---

## 📐 DIAGRAM 2: Avatar Animation Pipeline (Supplementary)

### Layout Type
**Vertical Flow Diagram** (smaller, can be inset)

### Components (Top to Bottom):

1. **Audio Input**
   - Source: TTS output
   - Format: WAV stream

2. **Audio Analysis**
   - Component: RMS Calculator
   - Output: Amplitude envelope (0.0-1.0)

3. **Animator Bridge**
   - Component: Unity Animator Controller
   - Inputs:
     - `MouthOpen` (float 0-1)
     - `IsListening` (bool)
     - `IsSpeaking` (bool)
   - Blend Tree: 10 blend shapes

4. **Render Output**
   - Components (parallel):
     - Mesh Deformation (lip sync)
     - LED State Machine (color transitions)
     - Procedural Head IK (look direction)

5. **Final Avatar State**
   - Visual: Rendered robot character
   - Frame rate: 72 FPS (VR target)

---

## 📐 DIAGRAM 3: Memory Architecture (Technical Detail)

### Layout Type
**Stacked Bar Chart** showing VRAM allocation

### Sections (from bottom to top):
1. **Unity Rendering** (2.1GB)
   - Textures, meshes, shaders
   - Color: Purple

2. **Whisper Model** (0.5GB)
   - Persistent in VRAM
   - Color: Light Blue

3. **Ollama Inference** (4.1GB)
   - Active during generation
   - Color: Orange

4. **Free Buffer** (1.3GB)
   - Overhead for OS/drivers
   - Color: Gray

**Total**: 8GB VRAM (typical consumer GPU)

### Annotations:
- Red line at 8GB mark (hardware limit)
- Green checkmark: "Optimized for RTX 3060 Ti"
- Note: "Dynamic allocation - not all loaded simultaneously"

---

## 📐 DIAGRAM 4: Communication Protocol (Sequence Diagram)

### Layout Type
**UML Sequence Diagram** (optional, for technical appendix)

### Actors (vertical lifelines):
1. Unity Client
2. FastAPI Server
3. Ollama Runtime

### Message Sequence:
```
Unity → FastAPI: WebSocket Connect
FastAPI → Unity: ACK (connection established)

Unity → FastAPI: POST /rag/ask {"query": "..."}
FastAPI → Ollama: HTTP POST /api/generate
Ollama → FastAPI: Stream tokens {"response": "...", "done": false}
FastAPI → Unity: WebSocket message {"status": "generating"}
Ollama → FastAPI: Final token {"done": true}
FastAPI → Unity: WebSocket message {"response": "...", "complete": true}
Unity → Unity: Trigger TTS + Animation
```

**Timing Annotations**:
- Vertical bars showing duration of each operation
- Total time: ~800ms (highlighted)

---

## 🎨 Drawing Tool Recommendations

### Recommended Software:
1. **Draw.io** (diagrams.net) - Free, web-based
   - Template: "Software Architecture"
   - Export: PNG (high-res) or SVG

2. **Lucidchart** - Professional tool
   - Template: "Cloud Architecture" or "Network Diagram"

3. **Figma** - For polished graphics
   - Use component libraries for tech icons

4. **Mermaid** (code-based) - For version control
   - Can be rendered in GitHub markdown
   - Example starter:
     ```mermaid
     graph LR
         A[User] -->|Voice| B[Unity]
         B -->|WebSocket| C[Python Backend]
         C -->|HTTP| D[Ollama]
     ```

### Icon Libraries:
- **Font Awesome** (free): https://fontawesome.com/
- **Material Icons**: https://fonts.google.com/icons
- **Devicon** (tech logos): https://devicon.dev/

---

## 📏 Final Tips for Polished Diagrams

1. **Consistency**: Use same rounded corner radius (8px) throughout
2. **Spacing**: Maintain 40px padding between major components
3. **Typography**: 
   - Titles: 16px Bold (e.g., Roboto)
   - Labels: 12px Regular
   - Technical details: 10px Monospace (e.g., Courier)
4. **Alignment**: Use grid snapping (10px or 20px grid)
5. **Shadows**: Subtle drop shadows (2px offset, 10% opacity) for depth
6. **Arrows**: Use 3px stroke width for main flow, 1.5px for secondary
7. **Export**: PNG at 2x resolution (2400×1600px) for crisp GitHub rendering

---

## 🖼️ Placement in README

### Suggested Locations:

1. **DIAGRAM 1 (System Architecture)**: 
   - After "## 🏗️ System Architecture" section
   - Replaces `[INSERT ARCHITECTURE DIAGRAM HERE]` placeholder

2. **DIAGRAM 2 (Avatar Pipeline)**:
   - After "## 🎨 Avatar Design" section
   - Or in a collapsed `<details>` section for technical readers

3. **DIAGRAM 3 (Memory)**:
   - In "## 🔬 Technical Highlights" under "Memory-Optimized LLM Inference"

4. **DIAGRAM 4 (Sequence)**:
   - Optional: In technical appendix or linked documentation

---

**Good luck with the diagrams! Focus on DIAGRAM 1 as the hero image - it's the most important for showcasing system complexity.** 🚀

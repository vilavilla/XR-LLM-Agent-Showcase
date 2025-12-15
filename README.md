# 🤖 Real-Time Embodied AI Agent in XR
### *Bachelor's Thesis Project - UPC Barcelona (2024-2025)*

[![Status](https://img.shields.io/badge/Status-In%20Development-yellow)]()
[![Unity](https://img.shields.io/badge/Unity-2021.3+-000000?logo=unity)]()
[![Python](https://img.shields.io/badge/Python-3.9+-3776AB?logo=python&logoColor=white)]()
[![License](https://img.shields.io/badge/License-Academic-blue)]()

> **A low-latency, fully offline conversational AI system running on consumer hardware, featuring real-time speech processing, local LLM inference, and procedural avatar animation in Mixed Reality.**

[INSERT DEMO GIF HERE - Full interaction loop: wake word → speech → avatar response]

---

## 🎯 Project Overview

This project demonstrates the integration of **modern Large Language Models** with **real-time XR applications** on **consumer-grade hardware**, addressing critical challenges in:

- **Latency Optimization**: Achieving <200ms perceived response time for conversational AI
- **Resource Constraints**: Running quantized 7B-parameter models on limited VRAM
- **System Architecture**: Orchestrating multi-process pipelines with WebSocket communication
- **Procedural Animation**: Generating lip-sync and facial expressions from audio in real-time

**Key Achievement**: Complete offline execution without cloud dependencies, suitable for privacy-sensitive applications (healthcare training simulation).

---

## 🏗️ System Architecture

### High-Level Pipeline

```
User Voice Input → STT (Whisper) → LLM (Mistral/Llama3) → TTS (Piper) → Procedural Animation → XR Output
         ↓              ↓                    ↓                  ↓                ↓
    Microphone    Python Server     Ollama Inference    Audio Synthesis    Unity Avatar
```

### Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Frontend** | Unity 2021.3 (C#) | XR rendering, avatar control, user interaction |
| **Backend** | Python 3.9 + FastAPI | Inference orchestration, RAG system |
| **LLM Runtime** | Ollama (GGUF quantized) | Local model inference (Mistral 7B / Llama3 8B) |
| **Speech-to-Text** | Whisper.cpp (base model) | Offline audio transcription (~67ms load time) |
| **Text-to-Speech** | Piper TTS (neural) | Low-latency voice synthesis (Spanish voices) |
| **Communication** | WebSockets (ws://) | Bi-directional real-time messaging |
| **Wake Word** | Windows Speech Recognition | Hands-free activation ("Hey Marc") |
| **Avatar Rendering** | Custom PBR Shaders | Robotic character with LED feedback systems |

[INSERT ARCHITECTURE DIAGRAM HERE]

---

## 🔬 Technical Highlights

### 1. **Memory-Optimized LLM Inference**
- **Challenge**: Running 7B parameter models on 8GB VRAM (consumer GPUs)
- **Solution**: 
  - GGUF 4-bit quantization (reduces model size from ~14GB to ~4GB)
  - Ollama runtime with context window management (4096 tokens)
  - Streaming responses to reduce perceived latency
- **Result**: Stable inference at ~30 tokens/sec on RTX 3060 Ti

### 2. **Low-Latency Multi-Process Communication**
- **Challenge**: Coordinating Python AI stack with Unity game loop (60 FPS requirement)
- **Solution**:
  - Asynchronous WebSocket server (FastAPI + uvicorn)
  - Unity-side non-blocking coroutines with message queuing
  - Heartbeat system for connection monitoring
- **Result**: <50ms network overhead, ~200ms end-to-end response time

### 3. **Procedural Avatar Animation System**
- **Challenge**: Synchronizing lip movements with synthesized speech without pre-recorded animations
- **Solution**:
  - RMS-based audio analysis for mouth open/close states
  - Unity Animator parameter blending (10 blend shapes)
  - LED feedback system driven by conversation state machine
  - Head tracking with procedural IK for natural eye contact
- **Result**: Believable character presence with zero manual animation work

### 4. **Offline-First Architecture**
- **Challenge**: Healthcare data privacy requirements (no cloud APIs)
- **Solution**:
  - All models run locally (Whisper, Mistral, Piper)
  - RAG system using local vector database (ChromaDB)
  - Medical documentation indexed from PDFs (sentence-transformers)
- **Result**: Zero external API calls, GDPR-compliant by design

---

## 📊 Performance Metrics

| Metric | Target | Achieved | Hardware |
|--------|--------|----------|----------|
| **End-to-End Latency** | <500ms | ~200ms | RTX 3060 Ti, 32GB RAM |
| **LLM Inference Speed** | >20 tok/s | ~30 tok/s | Mistral 7B Q4_K_M |
| **STT Transcription** | <2s | ~500-1000ms | Whisper base model |
| **TTS Synthesis** | <300ms | ~143ms | Piper (24 words) |
| **VRAM Usage** | <8GB | ~5.2GB | Peak during inference |
| **Frame Rate (VR)** | 72 FPS | 72 FPS | Meta Quest 2 target |

---

## 🎨 Avatar Design

The robotic avatar ("Toro") features:
- **Physically-Based Rendering**: Metallic/roughness workflow for realistic materials
- **Reactive LED System**: 2 LED strips that change color based on conversation state:
  - 🔵 Blue (Idle): Ready to listen
  - 🟠 Orange (Listening): Processing user input
  - 🟢 Green (Speaking): Delivering response
- **Procedural Animation**:
  - Idle breathing motion (sine wave displacement)
  - Head tilting correlated with audio RMS
  - Lip-sync driven by Animator blend trees
- **Design Philosophy**: Deliberately non-humanoid to avoid uncanny valley, inspired by industrial robots

[INSERT AVATAR RENDER/SCREENSHOT HERE]

---

## 🧪 Use Case: Medical Training Simulation

**Context**: Developed in collaboration with the **Donation and Transplant Institute (DTI)** for training healthcare professionals in organ donation protocols.

**Features**:
- **Retrieval-Augmented Generation (RAG)**: LLM queries enriched with medical guidelines from PDF documents
- **Session Persistence**: Clinical context maintained across conversations
- **Privacy-First**: PII detection and automatic sanitization
- **Multilingual**: Spanish/Catalan/English support via Whisper

**Example Interaction**:
```
User: "¿Cuáles son los criterios de donación renal?"
System: [Searches DTI guidelines] → "Los criterios principales incluyen..."
Avatar: [Speaks response with procedural gestures]
```

---

## 🛠️ Development Challenges & Solutions

### Challenge 1: Unity-Python Interoperability
**Problem**: Unity's .NET runtime and Python's asyncio don't naturally communicate.

**Solution**:
- Created custom C# WebSocket client with async/await pattern
- Implemented message serialization with Newtonsoft.Json
- Built reconnection logic with exponential backoff

### Challenge 2: Real-Time Audio Processing
**Problem**: Unity's Microphone API provides raw PCM data, but Whisper expects 16kHz WAV.

**Solution**:
- Implemented downsampling filter in C# (44.1kHz → 16kHz)
- Buffered audio in circular queue to handle variable-length speech
- Added VAD (Voice Activity Detection) to auto-segment recordings

### Challenge 3: VRAM Management
**Problem**: Competing processes (Unity, Ollama, Whisper) fighting for GPU memory.

**Solution**:
- Load Whisper model once at startup, keep in VRAM
- Configure Ollama with `num_gpu=24` layers (partial offloading)
- Unity shader LOD system to reduce VRAM pressure

---

## 📐 System Requirements

### Minimum (Development)
- **CPU**: Intel i5-9400 / AMD Ryzen 5 3600
- **GPU**: NVIDIA RTX 2060 (6GB VRAM)
- **RAM**: 16GB DDR4
- **Storage**: 10GB free space (models + project)

### Recommended (Optimal Performance)
- **CPU**: Intel i7-12700K / AMD Ryzen 7 5800X
- **GPU**: NVIDIA RTX 3060 Ti (8GB VRAM)
- **RAM**: 32GB DDR4
- **Storage**: SSD with 15GB free space

### XR Target Device
- **Meta Quest 2** (Standalone mode not supported - requires PC tethering via Oculus Link)

---

## 🚀 Quick Start

> **Note**: Full source code will be released after thesis defense (June 2025) due to university IP policies.

### Prerequisites
1. **Install Ollama**: [https://ollama.com/](https://ollama.com/)
2. **Pull LLM Model**: `ollama pull mistral`
3. **Python 3.9+** with virtual environment support

### Setup (High-Level)
```bash
# 1. Clone repository (after public release)
git clone https://github.com/[username]/XR-LLM-Agent-Showcase.git

# 2. Install Python dependencies
cd Backend
pip install -r requirements.txt

# 3. Start inference backend
python run_server.py  # Runs on http://127.0.0.1:8000

# 4. Open Unity project
# Open Unity Hub → Add Project → Select project folder
# Scene: Assets/VRConversationalAI/Scenes/MainScene.unity
# Press Play (backend auto-starts if configured)

# 5. Test voice interaction
# Say "Hey Marc" → Ask a question → Observe avatar response
```

---

## 📚 Documentation

### Core Documentation
- **[QUICKSTART_VOICE_SYSTEM.md](QUICKSTART_VOICE_SYSTEM.md)**: 15-minute setup guide for voice pipeline
- **[VOICE_SYSTEM_ARCHITECTURE.md](docs/VOICE_SYSTEM_ARCHITECTURE.md)**: Detailed technical architecture
- **[TESTING_GUIDE.md](docs/TESTING_GUIDE.md)**: Component testing procedures
- **[TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)**: Common issues and solutions

### API Documentation
- **Interactive Docs**: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) (Swagger UI)
- **Backend Architecture**: [docs/Architecture.md](docs/Architecture.md)

---

## 🎓 Academic Context

**Institution**: Facultat d'Informàtica de Barcelona (FIB-UPC)  
**Program**: Bachelor's Degree in Computer Science  
**Duration**: September 2024 - June 2025  
**Collaboration**: Instituto de Donación y Trasplante (DTI)  

**Research Focus**:
- Human-Computer Interaction in XR environments
- Efficient deployment of LLMs on edge devices
- Embodied conversational agents for specialized domains

**Supervisor**: [To be filled]  
**Defense Date**: June 2025 (expected)

---

## 🔮 Future Work

### Planned Improvements
- [ ] **Interrupt Handling**: Allow users to interrupt long AI responses
- [ ] **Emotion Detection**: Analyze user sentiment to modulate avatar behavior
- [ ] **Multi-Modal Input**: Gesture recognition + voice commands
- [ ] **Standalone Quest Build**: Optimize for on-device inference (requires model distillation)
- [ ] **Benchmark Suite**: Automated performance regression testing

### Research Extensions
- Explore **function calling** for action-oriented interactions (e.g., "Show me the protocol PDF")
- Investigate **vision-language models** (e.g., LLaVA) for visual context understanding
- Test **speculative decoding** for 2x inference speedup

---

## 🙏 Acknowledgments

### Technologies
- **[Ollama](https://ollama.com/)**: Local LLM runtime
- **[Whisper.cpp](https://github.com/ggerganov/whisper.cpp)**: OpenAI Whisper port by ggerganov
- **[Whisper.unity](https://github.com/Macoron/whisper.unity)**: Unity integration by Macoron
- **[Piper TTS](https://github.com/rhasspy/piper)**: Neural TTS by Rhasspy
- **[ollama-unity](https://github.com/Haoming02/ollama-unity)**: Architectural inspiration

### Inspiration
This project stands on the shoulders of giants in the open-source AI community, particularly:
- The **Hugging Face** team for democratizing LLM access
- **ggerganov** for making Whisper run on CPUs
- **Meta Reality Labs** research on embodied AI

---

## 📧 Contact

**Author**: Joan V.  
**LinkedIn**: [To be filled]  
**Email**: [Academic email]  

**Interested in this project?**  
I'm actively seeking **research internships** in:
- Embodied AI / Conversational Agents
- Real-Time Graphics & XR Systems
- Human-Computer Interaction

📍 **Open to opportunities in**: Switzerland, EU, Remote

---

## 📄 License

This project is part of an academic thesis. Source code will be released under [MIT License] after university defense (June 2025).

**Current Status**: 🔒 Closed source (IP restrictions)  
**Post-Defense**: 🔓 Open source planned

---

<div align="center">

**Built with ❤️ for the future of Human-AI Interaction in XR**

FIB-UPC

</div>

## ⚖️ License & Copyright

**© 2025 [Tu Nombre Completo]. All Rights Reserved.**

This repository contains the documentation and architectural overview of a Bachelor Thesis (TFG) currently in progress at **Universitat Politècnica de Catalunya (UPC - FIB)**.

Due to academic regulations and intellectual property protection prior to the thesis defense (expected **July 2025**), the source code is currently **private**.

* 🚫 **No permission is granted** to use, copy, modify, or distribute the contents of this repository without explicit written permission from the author.
* 👀 This repository is intended solely for **portfolio and demonstration purposes**.

The full source code will be released under an open-source license (MIT/Apache 2.0) *after* the successful defense of the thesis.

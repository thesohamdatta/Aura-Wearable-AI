<h1 align="center">Aura</h1>

<p align="center">
  <em>A wearable AI pendant that remembers what you see and hear.</em><br/>
  <em>Open source. Always with you.</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/license-MIT-black?style=flat-square" />
  <img src="https://img.shields.io/badge/platform-ESP32--S3-orange?style=flat-square" />
  <img src="https://img.shields.io/badge/battery-12h-brightgreen?style=flat-square" />
  <img src="https://img.shields.io/badge/weight-45g-blue?style=flat-square" />
  <img src="https://img.shields.io/badge/cost-$70-purple?style=flat-square" />
</p>

<p align="center">
  <a href="https://auraos.framer.ai/">Website</a>
</p>

---

<p align="center">
  <img src="https://github.com/user-attachments/assets/f3c6feb5-d5c1-4739-a034-eb468905ccef" width="30%" alt="Aura pendant side view" />
  &nbsp;
  <img src="https://github.com/user-attachments/assets/56118e77-75c5-4ce1-bdb6-11a28883b16d" width="30%" alt="Aura ESP32-S3 board" />
  &nbsp;
  <img width="972" height="310" alt="Aura overview" src="https://github.com/user-attachments/assets/d564ad2a-e63e-46ed-b918-5a199e311106" />
</p>

---

## What It Does

Aura is a wearable memory assistant. You can wear it as a pendant, clip, microphone, or watch-style device. It captures conversations, ideas, faces, and scenes throughout your day so you can ask about them later.

```text
"Where did I park?"               -> Aura knows.
"What did mom ask me to buy?"     -> Milk, eggs, and whatever else you forgot.
"Did I take my meds this morning?"-> Yes. Relax.
```

No screen. No friction. Less distraction.

---

## Hardware Stack

<p align="center">
  <img src="https://github.com/user-attachments/assets/877e66fb-ac78-4f53-8ce2-252a449bdf52" width="75%" alt="Hardware diagram" />
</p>

<div align="center">

| Component | Spec | Role |
|---|---|---|
| **XIAO ESP32-S3 Sense** | Dual-core 240MHz, 8MB PSRAM | Main controller |
| **OV2640 Camera** | 2MP, 66 deg FOV, JPEG output | Visual capture |
| **I2S MEMS Microphone** | 16kHz, omnidirectional | Audio capture |
| **LiPo Battery Pack** | 400mAh + 200mAh | Portable power |
| **3D-Printed Enclosure** | PLA/PETG | Wearable housing |
| **30 AWG Wires** | Flexible, ultra-thin | Internal wiring |

</div>

> **Estimated BOM:** ~$30 | **Weight:** ~45g | **Battery life:** 5 to 8 hours

---

## Software Stack

<p align="center">
  <img src="https://github.com/user-attachments/assets/b6ac55a4-0b20-4d09-aec3-f96c68d22611" width="75%" alt="Software stack" />
</p>

<div align="center">

| Layer | Technology | Purpose |
|---|---|---|
| **Firmware** | C++, Arduino, ESP-IDF | Device control, capture, BLE, Wi-Fi |
| **Mobile App** | Flutter, Dart | iOS and Android interface |
| **Backend API** | Python 3.11, FastAPI | Processing, routing, storage |
| **Database** | Firebase, Pinecone | Memories and vector search |
| **Communication** | BLE 5.0, Wi-Fi 802.11n | Device to app to cloud |

</div>

---

## Data stack


<p align="center">
  <img src="https://github.com/user-attachments/assets/bf38c30c-c303-4928-8b87-f6bba0ac83f2" width="75%" alt="AI pipeline" />
</p>

```text
Audio -> Transcription -> LLM processing -> Memory + Search
Image -> Vision model -> /
```

<div align="center">

| Provider | Role | Mode |
|---|---|---|
| **Groq / Llama** | Real-time transcription and reasoning | Cloud, fast |
| **OpenAI GPT-4o** | Vision understanding | Cloud, high quality |
| **Ollama** | Fully local inference | Local, private |
| **Deepgram** | Streaming transcription | Cloud, real-time |
| **Whisper** | Offline fallback | Local, offline |

</div>

If you do not want cloud inference, you can run the stack locally with Ollama.

---

## Build It Yourself

```bash
# 1. Clone the repository
git clone https://github.com/thesohamdatta/Aura-Wearable-AI.git
cd Aura-Wearable-AI

# 2. Flash firmware to your ESP32-S3
cd firmware
platformio run -e seeed_xiao_esp32s3 --target upload

# 3. Configure your environment
cp .env.template .env
# Add: GROQ_API_KEY / OPENAI_API_KEY / OLLAMA_URL

# 4. Start the backend
cd backend
pip install -r requirements.txt
python main.py
```

---

## Usage

<p align="center">
  <img src="./assets/app-walkthrough.gif" width="60%" alt="App walkthrough" />
</p>

1. Wear it.
2. Let Aura capture in the background.
3. Ask questions in the app.
4. Review transcripts, summaries, and action items.

**LED status**

<div align="center">

| Light | Meaning |
|---|---|
| Green | Recording active |
| Blue | Syncing to app |
| Yellow | Low battery |
| Off | Standby / idle |

</div>

---

## Contributing

We are building wearable AI in the open, and contributions are welcome.

```text
Found a bug?      -> Open an issue
Have an idea?     -> Start a discussion
Want to build?    -> Fork -> Branch -> PR
Improve docs?     -> Docs are code too
Other language?   -> Help translate
```

---

## Team

Built by people who believe wearable AI should be open, affordable, and under your control.

**Core Team:** [Soham Datta](https://www.linkedin.com/in/thesohamdatta) and [Laxman Pajai](https://www.linkedin.com/in/laxman-pajai-434b98239/?originalSubdomain=in)

---

<p align="center">
  <strong>Always with you. Always for you.</strong><br/>
  <sub>Star the repo if this project matters to you.</sub>
</p>

---

<sub>

**Disclaimer:** This project was developed with help from Claude Code, Antigravity, and Based Hardware (OMI) resources as inspiration, along with a lot of learning and a lot of late nights.

</sub>

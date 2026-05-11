# 🚀 Face Attendance Embedded System

A real-time face attendance system built with **Desktop GUI**, **REST API**, **PostgreSQL** và **React Dashboard**.

The system supports real-time facial recognition from camera streams, automatic attendance logging, web-based monitoring dashboard, and REST API/WebSocket communication.

![Python](https://img.shields.io/badge/Python-3.10+-blue)
![FastAPI](https://img.shields.io/badge/FastAPI-0.110+-green)
![React](https://img.shields.io/badge/React-18-61DAFB)
![Docker](https://img.shields.io/badge/Docker-required-2496ED)

---

## 🏗️ System Architecture

```
┌──────────────┐   WebSocket    ┌──────────────────┐
│  GUI (PyQt6) │ ─────────────► │ API (FastAPI)    │
│  Camera Feed │ ◄───────────── │ Face Recognition │
└──────────────┘   bbox/names   │ Attendance Logic │
                                └────────┬─────────┘
                                         │ asyncpg
                                ┌────────▼─────────┐
                                │  PostgreSQL DB   │
                                │  (Docker)        │
                                └────────┬─────────┘
                                         │ REST API
                                ┌────────▼─────────┐
                                │  React Dashboard │
                                │  localhost:5173  │
                                └──────────────────┘
```

---

## ✨ Main Features

🎯 Realtime Face Recognition
- Real-time facial recognition
- Webcam / IP camera support
- Bounding box + name visualization
- Multi-face detection support
---
📋 Attendance Management
- Automatic attendance logging
- PostgreSQL attendance storage
- Duplicate attendance prevention
- Check-in time tracking
---
🖥️ Desktop GUI (PyQt6)
- Live camera feed
- Start / Stop inference
- Camera settings
- Realtime monitoring
---
🌐 REST API + WebSocket
- FastAPI backend
- REST endpoints
- WebSocket realtime streaming
- Swagger API documentation
---
📊 React Dashboard
- Attendance history viewer
- Dashboard statistics
- Log management
- Realtime monitoring interface
---
# ⚙️ System Requirements

| Component | Version |
|---|---|
| Python | 3.10+ |
| Node.js | 18+ |
| Docker Desktop | Latest |
| PostgreSQL | 15+ |
| GPU (optional) | CUDA 11.8+ |
---
## ⚙️ Setting

### 1. Clone repository

```bash
git clone https://github.com/levanmanh2901/face-attendance-embedded-system.git
cd face-attendance-embedded-system
```

### 2. Add Face Images for Recognition

Place face images inside the `assets/faces/` directory. **Filename = person's name**.

```
assets/faces/
├── NguyenVanA.jpg
├── TranThiB.jpg
└── ...
```

> Each person only needs one image with a clear face and good lighting conditions.

### 3. Start PostgreSQL (Docker)

```bash
docker compose up -d
```

Check if the database is ready:
```bash
docker compose ps
```

### 4. Install Python Dependencies
```bash
pip install -r requirements.txt
```

> **GPU:** Replace `onnxruntime` with `onnxruntime-gpu` in `requirements.txt` if CUDA is available.
>
> **Windows + Conda:** It is recommended to install dependencies using `python -m pip install -r requirements.txt` after running `conda activate <env>`, instead of using `pip --user`.
>
> **Important:** If Python prioritizes packages inside `C:\Users\<user>\AppData\Roaming\Python\...`, the `onnxruntime-gpu` package inside the Conda environment may be overridden by the CPU version of `onnxruntime`, causing the model to run on CPU even when CUDA is available.
>
> You can prevent this issue by using:
> ```bash
> conda env config vars set -n <env> PYTHONNOUSERSITE=1
> conda activate <env>
> python -m pip install -r requirements.txt
> python -c "import onnxruntime as ort; print(ort.__file__); print(ort.get_available_providers())"
> ```
>
> For fresh installations on Windows, it is recommended to use Python 3.11 for the project environment to reduce package compatibility issues.

### 5. Run the API Backend

```bash
python api.py
```

The API will start at `http://localhost:8000`.

Swagger documentation: `http://localhost:8000/docs`

### 6. Run the Desktop GUI

```bash
python gui.py
```

- Click **▶ Start** to enable the camera and begin face recognition
- Click **⚙ Settings** to change the model, camera source, or recognition threshold

### 7. Run the React Dashboard (Optional)

```bash
cd web
npm install
npm run dev
```

Open your browser at: `http://localhost:5173`
---

## Project Structure

```
face-reidentification/
├── api.py                 # FastAPI backend (REST + WebSocket)
├── gui.py                 # Desktop GUI (PyQt6)
├── db.py                  # PostgreSQL helpers (asyncpg)
├── main.py                # CLI entry point (without GUI)
│
├── models/                # SCRFD and ArcFace model wrappers
├── database/              # FAISS database implementation
├── utils/                 # Logging and helper utilities
│
├── weights/               # Model weights (.onnx) — excluded from Git
├── assets/
│   ├── faces/             # Face images for recognition
│   └── captures/          # Automatically saved cropped images — excluded from Git
│
├── web/                   # React dashboard (Vite)
│   ├── src/
│   │   ├── pages/         # Dashboard, Attendance, Unknowns, Settings
│   │   ├── api.js         # API service layer
│   │   └── App.jsx
│   └── package.json
│
├── init.sql               # PostgreSQL schema
├── docker-compose.yml     # PostgreSQL container configuration
└── requirements.txt
```

---

## PostgreSQL Connection Information

| Parameter | Default Value |
|---|---|
| Host | `localhost` |
| Port | `5432` |
| Database | `faceid_db` |
| Username | `faceid_user` |
| Password | `faceid_pass` |

Override the database connection using an environment variable:

```bash
DATABASE_URL=postgresql://user:pass@host:5432/dbname python api.py
```
---

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/settings` | Get current configuration |
| `POST` | `/api/settings` | Update configuration |
| `POST` | `/api/infer/start` | Start inference |
| `POST` | `/api/infer/stop` | Stop inference |
| `GET` | `/api/attendance` | Attendance history |
| `GET` | `/api/unknowns` | Unknown person logs |
| `GET` | `/api/stats` | Overall statistics |
| `POST` | `/api/attendance/log` | Save attendance log (multipart) |
| `POST` | `/api/unknown/log` | Save unknown person log (multipart) |
| `WS` | `/ws/infer` | Realtime recognition WebSocket |

---

## Troubleshooting

**Camera Cannot Be Opened:**
- Check the camera source in Settings (`0` = default webcam, or use an RTSP URL)

**ONNX Runtime / CUDA Issues:**
- Use `onnxruntime` (CPU version) if no GPU is available or CUDA is not properly installed
- Check error logs inside `app.log`
- If there are no visible errors but the model still runs on CPU, verify which runtime is being imported:

```bash
python -c "import onnxruntime as ort; print(ort.__file__); print(ort.get_available_providers())"
```

- If the result points to `AppData\Roaming\Python\...` instead of the Conda environment, the environment is being overridden by `user-site packages`
- The most stable solution is to create a new environment, enable `PYTHONNOUSERSITE=1`, and reinstall dependencies using `python -m pip`

---
**PostgreSQL Connection Failed:**
- Make sure Docker is running:

```bash
docker compose up -d
```

- Check running containers:

```bash
docker compose ps
```

---

**Web Dashboard Cannot Display Images:**
- Ensure the Vite development server is running (`npm run dev` inside the `web/` directory)
- Images are served through the `/captures` proxy → FastAPI

---

## References

- [SCRFD: Efficient Face Detection](https://github.com/deepinsight/insightface/tree/master/detection/scrfd)
- [ArcFace: Deep Face Recognition](https://github.com/deepinsight/insightface/tree/master/recognition/arcface_torch)
- [YOLOFace training guide](https://drive.google.com/drive/folders/1Df3xxfUsWDbMfqwTgOE7q2CeXakW4V8D?usp=sharing)
- [FAISS: Facebook AI Similarity Search](https://github.com/facebookresearch/faiss)

## 🚀 Future Improvements
- Face anti-spoofing
- Edge AI optimization
- ESP32 / Raspberry Pi integration
- Multi-camera management
- AI analytics dashboard
- Face mask recognition
- Mobile application
- AI Agent monitoring assistant

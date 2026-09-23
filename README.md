# Chat Trainer UI

A native Windows desktop app to control the gallery chat LoRA trainer on the
training PC. Built with **Tkinter** (Python 3.8) and packaged with **PyInstaller**
as a single `.exe` with a desktop icon.

The app is a thin client to the local **control server** (`trainer_control.py`),
which supervises `chat_trainer.py` (the trainer). It talks to
`http://127.0.0.1:8790`, so it works even if the box's firewall blocks the LAN.

## Features

- **Trainer tab** — live status (running / stopped / paused, last poll, since_id,
  trained pairs, idle, phase) and working **Pause / Resume / Stop / Restart AI /
  Train now** buttons.
- **Training progress** — a progress bar with step `N / total` and live loss,
  fed by the trainer's `ProgressCallback`.
- **Log** — scrolling tail of the trainer log.
- **Admin tab** — edit every trainer setting and save (restarts the trainer),
  plus a run-at-logon toggle.
- **Resilient** — if the control server is down it shows a banner + "Start
  server" button; startup errors are surfaced in a message box instead of a
  silent crash.

## Files

| File | Purpose |
|------|---------|
| `trainer_gui.py` | The Tkinter GUI (this is what becomes the EXE) |
| `trainer_control.py` | Web control server + trainer supervisor (`:8790`) |
| `chat_trainer.py` | The LoRA trainer (polls server, trains, uploads) |
| `trainer_gui.spec` | PyInstaller spec (reproducible one-file windowed build) |
| `trainer.ico` | App icon (16–256 px purple chat bubble) |
| `build_ui.bat` | Build script: PyInstaller + desktop shortcut |
| `run_chat_trainer.bat` | Launches the control server (logon task target) |
| `install_control.bat` | One-time: register at-logon task + start server |

## Install on the training PC

1. Install **Python 3.8.10** (64-bit) to `C:\Python38`.
2. Install the trainer's pinned deps (torch 1.13.1 CPU, transformers 4.40.2,
   peft 0.12.0, datasets 3.1.0, tokenizers 0.15/0.19, safetensors, etc.) —
   see the gallery-site `training/README.md` for exact versions (Win7-safe).
3. Install PyInstaller 4.10: `C:\Python38\python.exe -m pip install pyinstaller==4.10`
4. Copy these files to `C:\ai\`.
5. Run `install_control.bat` once (registers the `ChatTrainer` at-logon task
   and starts the control server).
6. Run `build_ui.bat` to build `C:\ai\dist\ChatTrainerUI.exe` and create the
   **"Chat Trainer"** desktop shortcut.

## Rebuild after a code change

```bat
copy trainer_gui.py trainer_gui.spec trainer.ico C:\ai\
C:\ai\build_ui.bat
```

Or manually:

```bat
C:\Python38\python.exe -m PyInstaller --clean --noconfirm trainer_gui.spec
```

## Config

Settings live in `C:\work\chat_trainer_config.json` (created on first start of
the control server). The trainer reads `config file > env var > built-in
default`. The bridge token is masked in the API/UI.

Optional: set `CONTROL_TOKEN` on `trainer_control.py` to require an
`X-Control-Token` header on the web API.
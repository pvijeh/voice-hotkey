# Voice Hotkey

A simple push-to-talk voice-to-text tool for Linux. Hold the INSERT key to record, release to transcribe and auto-type the text wherever your cursor is.

Uses OpenAI's Whisper for local speech recognition - no cloud API required.

## Requirements

- Linux (uses `xdotool` for typing)
- Python 3.8+
- A microphone

## Installation

1. Install system dependencies:

```bash
sudo apt install xdotool portaudio19-dev
```

2. Clone this repo:

```bash
git clone https://github.com/pvijeh/voice-hotkey.git
cd voice-hotkey
```

3. Install Python dependencies:

```bash
pip install -r requirements.txt
```

4. Install the script:

```bash
chmod +x voice-hotkey
cp voice-hotkey ~/.local/bin/
```

## Usage

```bash
voice-hotkey [--model MODEL]
```

Options:
- `--model`, `-m`: Whisper model to use (default: `base`)
  - `tiny` - Fastest, least accurate (~1GB RAM)
  - `base` - Good balance (~1GB RAM)
  - `small` - Better accuracy (~2GB RAM)
  - `medium` - High accuracy (~5GB RAM)
  - `large` - Best accuracy (~10GB RAM)

Controls:
- **Hold INSERT** - starts recording
- **Release INSERT** - transcribes and types the text at your cursor
- **Press ESC** - quit

## Autostart on Login

Create a systemd user service:

```bash
mkdir -p ~/.config/systemd/user

cat > ~/.config/systemd/user/voice-hotkey.service << 'EOF'
[Unit]
Description=Voice Hotkey Daemon (Hold INSERT to dictate)
After=graphical-session.target

[Service]
ExecStart=%h/.local/bin/voice-hotkey
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
EOF

systemctl --user daemon-reload
systemctl --user enable voice-hotkey
systemctl --user start voice-hotkey
```

To use a different model at startup, edit the service file and change the `ExecStart` line:

```
ExecStart=%h/.local/bin/voice-hotkey --model small
```

### Useful Commands

```bash
# Check status
systemctl --user status voice-hotkey

# View logs
journalctl --user -u voice-hotkey -f

# Stop/start
systemctl --user stop voice-hotkey
systemctl --user start voice-hotkey

# Disable autostart
systemctl --user disable voice-hotkey
```

## Customization

To change the hotkey, edit the script and change `keyboard.Key.insert` to another key:
- `keyboard.Key.f9`
- `keyboard.Key.ctrl_r`
- `keyboard.Key.scroll_lock`
- etc.

## How It Works

1. Listens for the INSERT key globally using `pynput`
2. Records audio from the microphone while the key is held
3. On release, sends audio to Whisper for transcription
4. Types the transcribed text using `xdotool`

## License

MIT

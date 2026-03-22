# ZeroClaw Guide: Android Setup with Termux

**March 22, 2026** — Simple Termux setup for the latest prebuilt ZeroClaw install with daemon persistence. Samsung Note 20 Ultra (aarch64) tested.

## ⚡ Quick Start
```bash
pkg update && pkg upgrade -y
pkg install git curl termux-api htop -y
mkdir -p ~/.cargo/bin
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

git clone https://github.com/zeroclaw-labs/zeroclaw.git
cd zeroclaw
./install.sh --prebuilt-only
cd ..
rm -rf zeroclaw

zeroclaw onboard
zeroclaw daemon
```

For reboot persistence and helper aliases, continue with Step 3.

## 📚 Other Guides
- `android-local-llm.md` - Run local LLMs on Android with Termux
- `android-cross-compile-guide.md` - Cross-compile Android binaries
- `setup-ssh-termux.md` - Set up SSH access for Termux

## 📱 Prerequisites
```
- Samsung Galaxy Note 20 Ultra (12GB RAM) / OR any Android Phone
- Termux (F-Droid version recommended, PlayStore OK)  
- Termux:Boot app (needed for auto-start after reboot)
- Telegram app
- OpenAI API key
```

## 🎯 Step 1: Termux Base Setup
```bash
pkg update && pkg upgrade -y
pkg install git curl termux-api htop -y
mkdir -p ~/.cargo/bin
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## 🛠️ Step 2: Install Latest Prebuilt ZeroClaw
```bash
# Clone repo and install latest prebuilt binary
git clone https://github.com/zeroclaw-labs/zeroclaw.git
cd zeroclaw
./install.sh --prebuilt-only
cd ..
rm -rf zeroclaw

# Verify
~/.cargo/bin/zeroclaw --version  # latest installed version
file ~/.cargo/bin/zeroclaw       # ELF 64-bit ARM aarch64
```

The installer places the `zeroclaw` binary in `~/.cargo/bin` automatically.

## ⚙️ Step 3: Aliases & Persistence
Add helper aliases, then create a Termux:Boot script so ZeroClaw starts again after reboot.
```bash
cat >> ~/.bashrc << 'EOF'
alias zeroclaw-stop="pkill -f zeroclaw"
alias zeroclaw-start="termux-wake-lock && nohup zeroclaw daemon > ~/.zeroclaw/daemon.out 2>&1 &"
alias zeroclaw-restart="zeroclaw-stop && sleep 3 && zeroclaw-start"
EOF

# Auto-start after reboot (requires Termux:Boot)
mkdir -p ~/.termux/boot

# Boot script
cat > ~/.termux/boot/zeroclaw << 'EOF'
#!/data/data/com.termux/files/usr/bin/sh
export PATH="$HOME/.cargo/bin:$PATH"
sleep 10
termux-wake-lock
~/.cargo/bin/zeroclaw daemon &
EOF
chmod +x ~/.termux/boot/zeroclaw

# Load the new aliases in the current shell
source ~/.bashrc
```

## 🔧 Step 4: Core Configuration
Run the onboarding wizard:
```bash
zeroclaw onboard
```

Recommended minimal setup
```
Provider: openai  (API key)
Channel: telegram (BotFather token) 
Users: * or YOUR_ID
Model: gpt-4o-mini
```

## ▶️ Step 5: Launch Production Daemon
```bash
zeroclaw-restart
tail -f ~/.zeroclaw/daemon.out  # "listening for message.."
```

## 🧪 Test Matrix
```
Telegram → "hello" → "Hello! I'm ZeroClaw..."
"Write Sabah weather scraper" → Python script delivered
Close Termux → Bot still replies  
Reboot phone → Auto-revives
```

Want to experiment with local LLMs on Android? See `android-local-llm.md`.

## 📊 Monitoring & Commands
```bash
zeroclaw --version      # Status
zeroclaw doctor         # Health  
zeroclaw-restart        # Restart
zeroclaw-stop           # Stop daemon
zeroclaw-start          # Start daemon
tail -f ~/.zeroclaw/daemon.out  # Logs
htop                    # 1 core idle (normal)
ps aux | grep zeroclaw  # PID check
```

## 🔒 Files Structure
```
~/.cargo/bin/zeroclaw           # Installed binary
~/.zeroclaw/config.toml         # OpenAI/Telegram keys
~/.zeroclaw/daemon.out          # Live logs
~/.zeroclaw/sessions/           # Chat history  
~/.termux/boot/zeroclaw         # Auto-start
```

## 🚨 Troubleshooting
| Symptom | Solution |
|---------|----------|
| Bot silent | `tail daemon.out` + `allowed_users=*` |
| Dies sleep | `zeroclaw-start` (has wake-lock) |
| htop 1 core | Android gating—loads under stress |
| `line 1: Not:` | Re-run `./install.sh --prebuilt-only` inside `zeroclaw/` |

## 📈 Performance (Note 20 Ultra)
```
Idle: 3-8mAh/hr, 1 core, 20MB RAM
Query: GPT-4o-mini <2s, full 8 cores
Uptime: 99.9% (wake-lock)
```

Reliable and lightweight for daily Android use. 🚀

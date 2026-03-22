# ZeroClaw Guide: Local LLM on Android with Termux

Back to the main setup guide: [README.md](README.md)

This guide covers the experimental local LLM path for ZeroClaw on Android. It is separate from the main setup because local models on phones are slower and more resource-constrained than hosted providers.

## Before You Start
What to expect:
```
- Better privacy: yes
- Lower API cost: yes
- Better speed: usually no
- Better quality on phone hardware: usually no
```

Why this is a separate guide:
```
- Local models on phones are much slower than hosted APIs
- Small default context windows can be too tight for ZeroClaw workloads
- Even simple messages can turn into large prompts after system context, memory, tools, and history are added
- Battery, heat, RAM pressure, and swap matter a lot on Android
```

## Prerequisites
```bash
pkg update && pkg upgrade -y
pkg install ollama -y
```

## Step 1: Start Ollama
```bash
ollama serve
```

Keep that shell running while Ollama is serving.

## Step 2: Pull A Small Model
Open another Termux session:
```bash
ollama pull gemma2:2b
```

You can also try:
```bash
ollama pull phi3:3b
```

## Step 3: Test Ollama First
Test the model before connecting ZeroClaw:
```bash
ollama run gemma2:2b
```

If this already feels too slow, ZeroClaw on top of it will usually feel slower.

## Step 4: Connect ZeroClaw To Ollama
Run:
```bash
zeroclaw onboard
```

Recommended values:
```
Provider: ollama
URL: http://localhost:11434
Model: gemma2:2b or phi3:3b
```

## Step 5: Validate The Setup
After onboarding, try a very small prompt first.

Examples:
```bash
zeroclaw agent -m "hello"
zeroclaw doctor
```

## Important Caveats
```text
- A 4k context window is often too small for ZeroClaw
- A simple "hi" can still become a large prompt after system context is added
- Small local models may feel slow and low quality for agent-style workloads
- Phone thermals can reduce performance during longer sessions
```

## Recommended Use
```text
- Use hosted providers for daily use
- Use local LLMs only for privacy-sensitive or offline experiments
- If possible, run Ollama on a stronger machine and point ZeroClaw at that remote instance instead of the phone
```

## Reality Check
If your goal is a responsive Telegram assistant on Android, hosted providers will usually perform much better than local models running on the phone itself.

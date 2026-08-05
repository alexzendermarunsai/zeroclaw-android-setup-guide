

# Guía de ZeroClaw: Configuración en Android con Termux

**22 de marzo de 2026** — Configuración simple en Termux para la instalación más reciente precompilada de ZeroClaw con persistencia del daemon. Probado en Samsung Note 20 Ultra (aarch64).

## ⚡ Inicio Rápido
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

Para persistencia tras reinicio y alias de ayuda, continúa con el Paso 3.

## 📚 Otras Guías
- [android-local-llm.md](android-local-llm.md) - Ejecuta modelos de lenguaje locales en Android con Termux
- [android-cross-compile-guide.md](android-cross-compile-guide.md) - Compilación cruzada de binarios para Android
- [setup-ssh-termux.md](setup-ssh-termux.md) - Configura acceso SSH para Termux

## 📱 Requisitos Previos
```
- Samsung Galaxy Note 20 Ultra (12GB RAM) / O cualquier teléfono Android
- Termux (se recomienda la versión de F-Droid, PlayStore está bien)  
- Aplicación Termux:Boot (necesaria para el inicio automático tras reinicio)
- Aplicación Telegram
- Clave API de OpenAI
```

## 🎯 Paso 1: Configuración Base de Termux
```bash
pkg update && pkg upgrade -y
pkg install git curl termux-api htop -y
mkdir -p ~/.cargo/bin
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## 🛠️ Paso 2: Instalar la Última Versión Precompilada de ZeroClaw
```bash
# Clona el repositorio e instala el binario precompilado más reciente
git clone https://github.com/zeroclaw-labs/zeroclaw.git
cd zeroclaw
./install.sh --prebuilt-only
cd ..
rm -rf zeroclaw

# Verifica
~/.cargo/bin/zeroclaw --version  # última versión instalada
file ~/.cargo/bin/zeroclaw       # ELF 64-bit ARM aarch64
```

El instalador coloca automáticamente el binario `zeroclaw` en `~/.cargo/bin`.

## ⚙️ Paso 3: Alias y Persistencia
Añade alias de ayuda, luego crea un script de Termux:Boot para que ZeroClaw se inicie nuevamente tras un reinicio.
```bash
cat >> ~/.bashrc << 'EOF'
alias zeroclaw-stop="pkill -f zeroclaw"
alias zeroclaw-start="termux-wake-lock && nohup zeroclaw daemon > ~/.zeroclaw/daemon.out 2>&1 &"
alias zeroclaw-restart="zeroclaw-stop && sleep 3 && zeroclaw-start"
EOF

# Inicio automático tras reinicio (requiere Termux:Boot)
mkdir -p ~/.termux/boot

# Script de inicio
cat > ~/.termux/boot/zeroclaw << 'EOF'
#!/data/data/com.termux/files/usr/bin/sh
export PATH="$HOME/.cargo/bin:$PATH"
sleep 10
termux-wake-lock
~/.cargo/bin/zeroclaw daemon &
EOF
chmod +x ~/.termux/boot/zeroclaw

# Carga los nuevos alias en la sesión actual
source ~/.bashrc
```

## 🔧 Paso 4: Configuración Principal
Ejecuta el asistente de incorporación:
```bash
zeroclaw onboard
```

Configuración mínima recomendada
```
Provider: openai  (clave API)
Channel: telegram (token de BotFather) 
Users: * o TU_ID
Model: gpt-4o-mini
```

## ▶️ Paso 5: Iniciar Daemon de Producción
```bash
zeroclaw-restart
tail -f ~/.zeroclaw/daemon.out  # "escuchando mensajes.."
```

## 🧪 Matriz de Pruebas
```
Telegram → "hello" → "¡Hola! Soy ZeroClaw..."
"Escribe un scraper del clima de Sabah" → Se entrega script en Python
Cerrar Termux → El bot sigue respondiendo  
Reiniciar teléfono → Se revive automáticamente
```

¿Quieres experimentar con modelos de lenguaje locales en Android? Consulta [android-local-llm.md](android-local-llm.md).

## 📊 Monitoreo y Comandos
```bash
zeroclaw --version      # Estado
zeroclaw doctor         # Estado de salud
zeroclaw-restart        # Reiniciar
zeroclaw-stop           # Detener daemon
zeroclaw-start          # Iniciar daemon
tail -f ~/.zeroclaw/daemon.out  # Registros
htop                    # 1 núcleo inactivo (normal)
ps aux | grep zeroclaw  # Verificación de PID
```

## 🔒 Estructura de Archivos
```
~/.cargo/bin/zeroclaw           # Binario instalado
~/.zeroclaw/config.toml         # Claves de OpenAI/Telegram
~/.zeroclaw/daemon.out          # Registros en vivo
~/.zeroclaw/sessions/           # Historial de chat  
~/.termux/boot/zeroclaw         # Inicio automático
```

## 🚨 Solución de Problemas
| Síntoma | Solución |
|---------|----------|
| Bot en silencio | `tail daemon.out` + `allowed_users=*` |
| Se apaga por inactividad | `zeroclaw-start` (incluye wake-lock) |
| htop 1 núcleo | Limitación de Android—carga bajo estrés |
| `line 1: Not:` | Vuelve a ejecutar `./install.sh --prebuilt-only` dentro de `zeroclaw/` |

## 📈 Rendimiento (Note 20 Ultra)
```
Inactivo: 3-8mAh/h, 1 núcleo, 20MB RAM
Consulta: GPT-4o-mini <2s, 8 núcleos completos
Tiempo activo: 99.9% (wake-lock)
```

Confiable y ligero para uso diario en Android. 🚀

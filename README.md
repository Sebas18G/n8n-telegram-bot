# 🤖 n8n Telegram AI Bot — Docker Setup

Bot de Telegram con agente de IA construido sobre **n8n**, con soporte para texto, imágenes y audio. Expuesto públicamente mediante **ngrok** y orquestado con **Docker Compose** + **Traefik**.

---

## 📐 Arquitectura

```
Telegram ──► n8n (Docker) ──► AI Agent (Ollama / Eleven Labs)
                │
              Traefik (reverse proxy + TLS)
                │
              ngrok (túnel público)
```

**Flujos principales:**
- `/start` → respuesta de bienvenida con sticker
- Mensajes de texto → AI Agent → respuesta por Telegram
- Imágenes → descarga y análisis con IA → respuesta
- Audios → descarga → Eleven Labs TTS → respuesta de audio

---

## 🗂 Estructura del repositorio

```
.
├── compose.yaml          # Docker Compose (n8n + Traefik)
├── .env                  # Variables de entorno (no subir a Git)
├── .env.example          # Plantilla de variables
├── .gitignore
├── agent.md              # System prompt del agente de IA
├── local-files/          # Archivos locales montados en n8n (/files)
└── README.md
```

---

## ⚙️ Requisitos

- [Docker](https://docs.docker.com/get-docker/) + Docker Compose
- [ngrok](https://ngrok.com/) (cuenta gratuita o de pago)
- Cuenta en [Telegram](https://telegram.org/) + bot creado con [@BotFather](https://t.me/BotFather)
- (Opcional) [Ollama](https://ollama.com/) corriendo localmente para el modelo LLM
- (Opcional) API key de [Eleven Labs](https://elevenlabs.io/) para síntesis de voz

---

## 🚀 Instalación

### 1. Clonar el repositorio

```bash
git clone https://github.com/tu-usuario/docker-n8n.git
cd docker-n8n
```

### 2. Configurar variables de entorno

```bash
cp .env.example .env
```

Edita `.env` con tus valores:

```dotenv
DOMAIN_NAME=ngrok-free.dev
SUBDOMAIN=tu-subdominio-ngrok
SSL_EMAIL=tu@email.com
GENERIC_TIMEZONE=America/Bogota
```

### 3. Crear el bot de Telegram

1. Abre Telegram y busca [@BotFather](https://t.me/BotFather)
2. Envía `/newbot` y sigue las instrucciones
3. Guarda el **token** que te entrega

### 4. Levantar los contenedores

```bash
docker compose up -d
```

n8n estará disponible en `http://localhost:5678`

### 5. Exponer con ngrok

```bash
ngrok http 5678
```

Copia la URL de **Forwarding** (ej: `https://xxxx.ngrok-free.dev`) y úsala como `WEBHOOK_URL` en n8n.

> ⚠️ **Nota:** El plan gratuito de ngrok cambia la URL en cada sesión. Para una URL fija, usa un plan de pago o configura un dominio propio.

### 6. Importar el workflow en n8n

1. Abre n8n en el navegador
2. Ve a **Workflows → Import**
3. Importa el archivo `template-workflow.json` (si existe) o crea los nodos manualmente siguiendo la arquitectura descrita

### 7. Configurar credenciales en n8n

Dentro de n8n, configura:
- **Telegram API**: pega el token del bot
- **Ollama** (si aplica): URL del servidor local (`http://host.docker.internal:11434`)
- **Eleven Labs** (si aplica): API key

---

## 🧠 Agente de IA (`agent.md`)

El archivo `agent.md` contiene el system prompt del agente. Puedes modificarlo para cambiar el comportamiento, tono y herramientas disponibles del bot.

---

## 🔄 Comandos útiles

```bash
# Ver logs en tiempo real
docker compose logs -f n8n

# Detener todo
docker compose down

# Actualizar imagen de n8n
docker compose pull && docker compose up -d

#En su defecto nomralmente se usa
docker compose up -d
```

---

## 🔒 Seguridad

- **Nunca** subas el archivo `.env` al repositorio. Ya está en `.gitignore`.
- Rota el token de Telegram si lo expones accidentalmente.
- Usa el archivo `.env.example` como referencia para otros colaboradores.

---

## 📝 Variables de entorno

| Variable | Descripción | Ejemplo |
|---|---|---|
| `DOMAIN_NAME` | Dominio base del túnel | `ngrok-free.dev` |
| `SUBDOMAIN` | Subdominio asignado por ngrok | `mi-tunel-abc` |
| `SSL_EMAIL` | Email para certificado TLS | `yo@email.com` |
| `GENERIC_TIMEZONE` | Zona horaria para cron nodes | `America/Bogota` |

---

## 🤝 Contribuir

1. Haz fork del repositorio
2. Crea una rama: `git checkout -b feature/mi-mejora`
3. Haz commit: `git commit -m "feat: descripción"`
4. Push: `git push origin feature/mi-mejora`
5. Abre un Pull Request
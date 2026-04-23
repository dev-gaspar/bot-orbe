# Bot Orbe — WhatsApp AI Sales Agent

> Agente conversacional multimodal para WhatsApp: texto, voz e imagen. Construido sobre Baileys + OpenAI Assistants + ElevenLabs.

[![Node.js](https://img.shields.io/badge/Node.js-22+-5FA04E?logo=node.js&logoColor=white)](https://nodejs.org/)
[![WhatsApp](https://img.shields.io/badge/WhatsApp-Baileys-25D366?logo=whatsapp&logoColor=white)](https://github.com/WhiskeySockets/Baileys)
[![OpenAI](https://img.shields.io/badge/OpenAI-Assistants_API-412991?logo=openai&logoColor=white)](https://platform.openai.com/docs/assistants/overview)
[![ElevenLabs](https://img.shields.io/badge/ElevenLabs-TTS-000000)](https://elevenlabs.io/)
[![Docker](https://img.shields.io/badge/Docker-ready-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![License: ISC](https://img.shields.io/badge/License-ISC-blue.svg)](#licencia)

---

## ¿Qué hace?

Atiende conversaciones comerciales en WhatsApp de forma autónoma con comportamiento similar a un asesor humano:

- 💬 **Texto** — responde usando un *assistant* de OpenAI con contexto persistente por usuario.
- 🎙️ **Voz** — transcribe audios entrantes con **Whisper** y puede responder con voz clonada vía **ElevenLabs**.
- 🖼️ **Imagen** — describe fotos del cliente con **GPT Vision** para cotizar productos a partir de capturas.
- 🧠 **Estado global** — cola de mensajes con concurrencia (50 hilos, timeout 20s) para escalar a múltiples clientes en paralelo.
- 📊 **API de telemetría** — reporta el estado del bot (`ready`, `require_action`, `auth_failure`) a un backend externo para monitoreo.

## Arquitectura

```
┌─────────────────┐      ┌──────────────────────────┐      ┌────────────────┐
│   WhatsApp      │◄────►│   Baileys Provider       │      │   MongoDB      │
│   (cliente)     │      │   + BuilderBot flows     │◄────►│   (historial)  │
└─────────────────┘      └────────────┬─────────────┘      └────────────────┘
                                      │
              ┌───────────────────────┼───────────────────────┐
              ▼                       ▼                       ▼
       ┌────────────┐         ┌───────────────┐        ┌────────────┐
       │  Whisper   │         │   OpenAI      │        │ ElevenLabs │
       │  (audio→   │         │   Assistants  │        │  (text→    │
       │   texto)   │         │   + Vision    │        │   voz)     │
       └────────────┘         └───────────────┘        └────────────┘
```

**Flujos implementados** (`src/flow/`):
- `flowWelcome` — primer contacto con el cliente
- `flowAssistant` — respuesta con OpenAI Assistants
- `flowListen` — ingesta y transcripción de notas de voz
- `flowVision` — análisis visual de imágenes enviadas

## Stack técnico

| Capa | Tecnología |
| --- | --- |
| Runtime | Node.js 22 (ES Modules) |
| Framework bot | [BuilderBot 1.1](https://www.builderbot.app/) |
| Proveedor WhatsApp | Baileys |
| LLM | OpenAI Assistants API + Vision |
| ASR | OpenAI Whisper |
| TTS | ElevenLabs |
| Media | `fluent-ffmpeg`, `sharp`, `ffmpeg-static` |
| DB | MongoDB |
| Deploy | Docker (alpine) |

## Quickstart

### 1. Clonar y configurar

```bash
git clone https://github.com/dev-gaspar/bot-orbe.git
cd bot-orbe
cp .env.example .env
# Edita .env con tus claves (OpenAI, ElevenLabs, URL del backend)
```

### 2. Correr en local

```bash
npm install
npm run dev
```

La primera vez escanea el QR con WhatsApp Web. A partir de ahí la sesión queda persistida.

### 3. Correr con Docker

```bash
docker build -t bot-orbe .
docker run -p 4000:4000 --env-file .env bot-orbe
```

## Variables de entorno

| Variable | Propósito |
| --- | --- |
| `PORT` | Puerto del servidor HTTP interno (default `4000`) |
| `URL_API` | Endpoint del backend que recibe telemetría del bot |
| `COOKIE_ADMIN_JWT` | Token de autenticación contra `URL_API` |
| `OPENAI_API_KEY` | API key de OpenAI (Assistants + Whisper + Vision) |
| `ASSISTANT_ID` | ID del *assistant* preconfigurado en OpenAI |
| `ELEVENLABS_API_KEY` | API key de ElevenLabs |
| `ELEVENLABS_VOICE_ID` | ID de la voz clonada a usar |

> ⚠️ **Nunca** commitees tu `.env` real. El archivo `.env.example` es solo una plantilla con nombres de variables.

## Límites de OpenAI (referencia)

El bot está calibrado para operar dentro de los límites de la API de Assistants:

- 200 000 tokens / minuto
- 500 peticiones / minuto
- 10 000 peticiones / día
- 2 000 000 tokens / día

La cola de BuilderBot (`concurrencyLimit: 50`) está dimensionada para no exceder esos límites con ~50 conversaciones simultáneas.

## Roadmap

- [ ] Panel de administración para editar flows sin redeploy
- [ ] Handoff a operador humano cuando baja la confianza del assistant
- [ ] Métricas en Prometheus / Grafana
- [ ] Soporte multi-número (pooling de sesiones de WhatsApp)

## Créditos

Fork de [`bambai-labs/bot-orbe`](https://github.com/bambai-labs/bot-orbe), presentado aquí como muestra de arquitectura de agentes conversacionales multimodales.

## Licencia

ISC — ver [`package.json`](./package.json).

---

**Autor:** [Jose Gaspar](https://devgaspar.me) · AI Engineer | Fullstack & DevOps

# Integraciones

## APIs HTTP Externas

| Dependencia | Proposito | Focalpoint |
|---|---|---|
| WAHA (WhatsApp HTTP API) | Recibir webhooks de WhatsApp y enviar texto, imágenes y documentos |  |
| Chatbot LangChain (`/v1/llm/question`) | Orquestación agéntica: interpreta el mensaje y decide cotizar, pagar o emitir |  |
| Qdrant | Fuente de conocimiento del agente (RAG: condicionado, FAQs, producto Rumbo) |  |
| Google Vertex AI (Gemini) | Transcripción de notas de voz (`audio/ogg`) a texto |  |
| SMTP Gmail | Envío del correo de bienvenida con el condicionado adjunto |  |

## Detalles de Integración

### WAHA — webhook de entrada

- **Dirección:** WAHA → `waha_gateway` `POST /waha/webhook`
- **Cuándo:** cada mensaje de la sesión configurada (`events: ["message"]`)
- **Contrato:** `event`, `session`, `payload` (`id`, `timestamp`, `from`, `fromMe`, `body`, `hasMedia`, `media.url`, `media.mimetype`)
- **Efecto:** el gateway normaliza el mensaje, transcribe audio si aplica, y lo entrega al LLM

### WAHA — `POST /api/sendText`

- **Quién:** `waha_gateway.send_waha_message` y `rumbia-backend.WahaService.enviar_mensaje`
- **Headers:** `Content-Type: application/json`, `X-Api-Key`
- **Body:** `{ "session", "chatId": "{numero}@c.us", "text", "reply_to", "linkPreview" }`
- **Cuándo:** respuesta del chatbot al usuario; también texto suelto desde el backend

### WAHA — `POST /api/sendImage`

- **Quién:** `WahaService.enviar_imagen_desde_base64`
- **Body:** `{ "session", "chatId", "file": { "mimetype", "filename", "data": "<base64>" }, "caption" }`
- **MIME:** `image/jpeg`, `image/png`, `image/gif`, `image/webp`
- **Cuándo:** JPEG de bienvenida generado a partir del HTML del correo, inmediatamente después de emitir la póliza

### WAHA — `POST /api/sendFile`

- **Quién:** `WahaService.enviar_documento`
- **Body:** `{ "session", "chatId", "caption", "file": { "mimetype", "filename", "data": "<base64>" } }`
- **MIME:** `application/pdf` (o Word si no hubo conversión)
- **Cuándo:** envío del condicionado particular al cliente por WhatsApp

### WAHA — `GET {media.url}`

- **Quién:** `speech2text.convert_speech_to_text`
- **Headers:** `X-Api-Key`
- **Cuándo:** el webhook trae `hasMedia` con `audio/ogg; codecs=opus`
- **Detalle:** se reemplaza el host `localhost` de la URL publicada por WAHA por `WAHA_API_URL` antes de descargar

### Chatbot LangChain — `POST /v1/llm/question`

- **Quién:** `waha_gateway` (flujo diseñado en `waha_router`)
- **Body:** `{ "user", "question", "assistant", "assistantName", "memory": true, "history": "", "conversationId" }`
- **Respuesta esperada:** `{ "status": "OK", "data": { "answer", "answerId", "conversationId" } }`
- **Timeout:** 30 segundos
- **Rol:** único orquestador agéntico; el gateway no llama al cotizador ni al backend de emisión

### Qdrant — fuente de conocimiento (RAG)

- **Quién:** el `llm-api` / Chatbot LangChain (fuera de este workspace). Ningún repo RumbIA abre cliente Qdrant
- **Uso:** retrieve de embeddings (condicionado, FAQs, producto Rumbo) antes de responder; upsert al indexar documentos
- **Relación:** el gateway solo habla con `CHATBOT_API_URL`; Qdrant queda detrás del agente

### Vertex AI Gemini — `models.generate_content`

- **Quién:** `waha_gateway.src.services.speech2text`
- **SDK:** `google-genai` con `vertexai=True`
- **Entrada:** bytes del audio + prompt de transcripción
- **Config:** `temperature=0.1`, `max_output_tokens=8192`, `response_mime_type=text/plain`
- **Salida:** texto plano que sustituye `payload.body`

### SMTP Gmail — envío de bienvenida

- **Quién:** `EmailService.enviar_email`
- **Host/puerto:** `SMTP_HOST` / `SMTP_PORT` (587 + STARTTLS)
- **Contenido:** HTML generado desde `BienvenidaRumbo.html` + adjunto PDF/Word
- **Cuándo:** al final de `PolizaService.emitir_poliza`, si se generó el documento

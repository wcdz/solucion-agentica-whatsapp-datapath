# Variables de entorno

Valores de ejemplo. No usar secretos reales en este archivo.

| Variable | Descripción | Valores |
|---|---|---|
| `LOG_LEVEL` | Nivel de logging del gateway y del backend | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` |
| `CHATBOT_API_URL` | URL base del chatbot LangChain que orquesta las herramientas | `http://localhost:8080`, `https://llm-api-xxxxx.run.app` |
| `ASSISTANT` | Identificador del asistente enviado en `POST /v1/llm/question` | UUID del asistente, ej. `84ae4421-0102-4ccc-9f17-5b9b12600324` |
| `ASSISTANT_NAME` | Nombre visible del asistente en el payload hacia el LLM | `RumbIA`, `Bot Test` |
| `WAHA_API_URL` | URL base del servicio WAHA (WhatsApp HTTP API) | `http://localhost:3000`, `https://waha-xxxxx.run.app` |
| `WAHA_API_KEY` | API key enviada en el header `X-Api-Key` hacia WAHA | string secreto, no versionar |
| `WAHA_SERVER` | URL de WAHA usada por el backend para envío de salida | misma familia que `WAHA_API_URL` |
| `GOOGLE_APPLICATION_CREDENTIALS` | Ruta al JSON de cuenta de servicio GCP para Vertex AI | `gcp-credentials.json` |
| `GCP_PROJECT_ID` | Proyecto GCP de Vertex AI | `is-geniaton-ifs-2025-g3` |
| `GCP_LOCATION` | Región de Vertex AI | `us-central1` |
| `VERTEX_AI_MODEL` | Modelo Gemini usado para transcribir audio | `gemini-2.0-flash-exp` |
| `APP_NAME` | Nombre de la aplicación backend | `RumbIA Backend` |
| `APP_VERSION` | Versión del backend | `1.0.0` |
| `DESCRIPTION` | Descripción de la API backend | `Backend orquestador de servicios para RumbIA - Agente Inteligente` |
| `HOST` | Host de bind del backend | `0.0.0.0` |
| `PORT` | Puerto HTTP. Cloud Run inyecta `PORT=8080` | `8000` (local), `8080` (Cloud Run), `8090` (gateway) |
| `DEBUG` | Modo debug de FastAPI | `True`, `False` |
| `RELOAD` | Auto-reload de uvicorn en desarrollo | `True`, `False` |
| `ALLOWED_ORIGINS` | Orígenes CORS | `["*"]` |
| `ALLOWED_METHODS` | Métodos HTTP CORS | `["GET","POST","PUT","DELETE","OPTIONS"]` |
| `ALLOWED_HEADERS` | Headers CORS | `["*"]` |
| `API_V1_PREFIX` | Prefijo de la API v1 del backend | `/api/v1` |
| `EXTERNAL_SERVICE_TIMEOUT` | Timeout en segundos hacia servicios externos | `30` |
| `MAX_RETRIES` | Reintentos hacia servicios externos | `3` |
| `SMTP_HOST` | Servidor SMTP del correo de bienvenida | `smtp.gmail.com` |
| `SMTP_PORT` | Puerto SMTP | `587` |
| `SMTP_USER` | Usuario SMTP | correo de la cuenta remitente |
| `SMTP_PASSWORD` | Contraseña de aplicación SMTP | secret, no versionar |
| `FROM_EMAIL` | Remitente del correo de póliza | mismo correo SMTP |
| `FROM_NAME` | Nombre visible del remitente | `RumbIA \| Bienvenido a Interseguro` |
| `CHROME_BIN` | Binario Chromium para html2image (Docker) | `/usr/bin/chromium` |
| `CHROMIUM_FLAGS` | Flags de Chromium headless | `--no-sandbox --disable-dev-shm-usage --headless` |

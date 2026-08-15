## /waha/webhook

```mermaid
sequenceDiagram
    participant WA as WhatsApp
    participant WH as WAHA
    participant GW as WAHA Gateway
    participant ST as speech2text
    participant VX as Vertex AI
    participant LLM as Chatbot LangChain
    participant MAP as waha_mapper

    WA->>WH: mensaje texto o audio
    WH->>GW: POST /waha/webhook
    GW->>MAP: is_group_or_broadcast(from)
    alt hasMedia y mimetype audio/ogg
        GW->>ST: convert_speech_to_text(media.url)
        ST->>ST: reescribe host a WAHA_API_URL
        ST->>WH: GET media (X-Api-Key)
        WH-->>ST: bytes OGG
        ST->>VX: generate_content (Gemini)
        VX-->>ST: transcripción
        ST-->>GW: texto
        GW->>GW: payload.body = transcripción
    end
    GW->>MAP: map_to_chatbot_payload
    GW->>LLM: POST /v1/llm/question
    LLM-->>GW: status OK + data.answer
    GW->>MAP: map_to_send_text_payload
    GW->>WH: POST /api/sendText
    WH->>WA: respuesta del agente
    GW-->>WH: 200 success
```

## /api/v1/rumbia/emision-poliza

```mermaid
sequenceDiagram
    participant P as Pasarela de pago
    participant API as rumbia.py
    participant PS as PolizaService
    participant DS as GenerateDocumentService
    participant ES as EmailService
    participant FS as html2image / Chromium
    participant WS as WahaService
    participant WH as WAHA
    participant SMTP as SMTP Gmail

    P->>API: POST /api/v1/rumbia/emision-poliza
    API->>PS: emitir_poliza(cliente, cotizacion)
    PS->>PS: siguiente ID y db/RumbIA###.json
    PS->>DS: generar_documento
    DS->>DS: leer plantilla Word y reemplazar «marcadores»
    DS->>DS: soffice convert-to pdf
    DS-->>PS: ruta PDF
    PS->>ES: generar_html_email
    ES->>ES: leer BienvenidaRumbo.html
    ES-->>PS: HTML
    PS->>FS: screenshot HTML a JPEG
    FS-->>PS: db/documentos/RumbIA###_email.jpg
    PS->>ES: enviar_email_bienvenida_poliza
    ES->>SMTP: STARTTLS + adjunto PDF
    SMTP-->>ES: ok
    PS->>WS: enviar_paquete_bienvenida_poliza
    WS->>WH: POST /api/sendImage (JPEG + caption)
    WS->>WH: POST /api/sendFile (PDF + caption)
    WH-->>WS: enviado
    PS-->>API: numero_poliza, email, whatsapp
    API-->>P: 201 EmisionPolizaResponse
```

## /api/v1/cotizaciones

```mermaid
sequenceDiagram
    participant C as Consumidor (LLM / cliente)
    participant R as cotizaciones.py
    participant S as CotizacionService
    participant X as Excel xlwings
    participant M as Cache memoria

    C->>R: POST /api/v1/cotizaciones
    R->>S: crear(cotizacion)
    S->>M: lookup hash edad/sexo/prima/periodo
    alt cache hit
        M-->>S: porcentaje, tasa, suma, devolucion
    else cache miss
        S->>X: abrir macro Parametros_Supuestos
        X-->>S: C11 C21 C10 C29 C30
        S->>M: guardar resultado
    end
    S->>S: armar tabla_devolucion JSON
    S-->>R: CotizacionResponse
    R-->>C: 201
```

## /api/v1/cotizaciones/coleccion

```mermaid
sequenceDiagram
    participant C as Consumidor (LLM / cliente)
    participant R as cotizaciones.py
    participant S as CotizacionService
    participant CFG as periodos_cotizacion.json
    participant X as Excel xlwings

    C->>R: POST /api/v1/cotizaciones/coleccion
    R->>S: crear_cotizacion_coleccion
    S->>CFG: periodos disponibles para la prima
    loop cada periodo
        S->>X: calcular cotización (o cache)
        X-->>S: detalle por periodo
    end
    S-->>R: CotizacionColeccionResponse
    R-->>C: 200
```

## /api/v1/cotizaciones/generar-imagen

```mermaid
sequenceDiagram
    participant C as Consumidor (LLM / cliente)
    participant R as cotizaciones.py
    participant I as ImageService
    participant S as CotizacionService
    participant FS as db/

    C->>R: POST /api/v1/cotizaciones/generar-imagen
    R->>I: generar_grafico_desde_endpoint
    I->>S: crear_cotizacion_coleccion
    S-->>I: cotizaciones por periodo
    I->>I: matplotlib gráfico + tabla JPEG 300 DPI
    I->>FS: cotizacion_prima{n}_edad{e}_{sexo}_{ts}.jpg
    I-->>R: ruta_archivo
    R-->>C: 201 ImageGenerationResponse
```

## /api/v1/cotizaciones/cache

```mermaid
sequenceDiagram
    participant C as Cliente
    participant R as cotizaciones.py
    participant S as CotizacionService

    C->>R: DELETE /api/v1/cotizaciones/cache
    R->>S: limpiar_cache_colecciones
    S-->>R: cantidad eliminada
    R-->>C: 200 mensaje + elementos_eliminados
```

## /api/v1/cotizaciones/cache/estadisticas

```mermaid
sequenceDiagram
    participant C as Cliente
    participant R as cotizaciones.py
    participant S as CotizacionService

    C->>R: GET /api/v1/cotizaciones/cache/estadisticas
    R->>S: obtener_estadisticas_cache
    S-->>R: counts de cache
    R-->>C: 200 estadisticas
```

## /api/v1/rumbia/saludo

```mermaid
sequenceDiagram
    participant C as Cliente
    participant API as rumbia.py

    C->>API: GET /api/v1/rumbia/saludo
    API-->>C: 200 message, agent_name RumbIA, status active
```

## /api/v1/rumbia/

```mermaid
sequenceDiagram
    participant C as Cliente
    participant API as rumbia.py

    C->>API: GET /api/v1/rumbia/
    API-->>C: 200 presentación del agente, status ready
```

## /api/v1/rumbia/health

```mermaid
sequenceDiagram
    participant C as Cliente
    participant API as rumbia.py

    C->>API: GET /api/v1/rumbia/health
    API-->>C: 200 status healthy, version 1.0.0
```

## /api/v1/rumbia/info

```mermaid
sequenceDiagram
    participant C as Cliente
    participant API as rumbia.py

    C->>API: GET /api/v1/rumbia/info
    API-->>C: 200 capacidades del orquestador (NLP, APIs, asistencia)
```

## /health

```mermaid
sequenceDiagram
    participant C as Cliente / orquestador
    participant API as FastAPI (backend o cotizador)

    C->>API: GET /health
    API-->>C: 200 status healthy
```

## /

```mermaid
sequenceDiagram
    participant C as Cliente
    participant API as FastAPI (backend o cotizador)

    C->>API: GET /
    API-->>C: 200 bienvenida, version y docs_url
```

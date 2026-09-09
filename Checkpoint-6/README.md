# Checkpoint 6: Ecosistemas Multimedia de Audio (Voice AI)
> **Curso:** AI Automation / Coderhouse  
> **Alumno:** Javier Vélez  
> **Repositorio:** `coderhouse-javier-velez/Checkpoint-6`

---

## 📌 Descripción del Proyecto

Este repositorio contiene la entrega formal correspondiente al **Checkpoint del Módulo 6 (Ecosistemas Multimedia de Audio - Voice AI)**. El objetivo del proyecto es evolucionar el circuito conversacional previamente desarrollado en el Módulo 5 (con arquitectura RAG) para incorporar una **capa nativa de interfaz de voz bidireccional (Voice AI)** de extremo a extremo en **n8n**.

El sistema permite recibir notas o archivos de voz, convertirlos a texto con alta velocidad mediante **OpenAI Whisper (STT)**, procesar la consulta a través de un **AI Agent** con base de conocimientos vectorial (RAG), y responder generando audio sintético de alta fidelidad con **ElevenLabs (TTS)**.

---

## 🛠️ Arquitectura del Workflow en n8n

El flujo de trabajo automatizado está integrado por los siguientes componentes clave:

```
[ Form Trigger / Webhook ]
            │ (Payload Binario: `data`)
            ▼
   [ OpenAI Whisper STT ] ────> (Idioma: `es`)
            │
            ▼
     [ Nodo IF (Check) ] ────> ¿Texto no vacío?
            │
      ┌─────┴─────┐
   (TRUE)      (FALSE) ──> [ Fin / Notificación Error ]
      │
      ▼
   [ AI Agent + RAG ] ────> (System Prompt: Máx. 200 caracteres)
            │
            ▼
   [ ElevenLabs TTS ] ────> (Model: Eleven Multilingual v2)
            │
            ▼
[ Respond to Webhook / Form ] ──> (Respuesta Audio Binario)
```

### Componentes y Configuración Técnica:

1. **Interceptación Binaria de Entrada (`n8n Form Trigger`):**
   * Configurado con un elemento de tipo `File` con la propiedad binaria etiquetada obligatoriamente como **`data`**.
   * Permite la carga directa de archivos de audio (`.mp3` / `.wav`) directamente desde la computadora para pruebas rápidas y eficientes.

2. **Oídos del Agente (`OpenAI Whisper` - Action: *Transcribe a recording*):**
   * **Input Binary Property:** `data`
   * **Options -> Language:** `es` (forzado a Español para acelerar la latencia de procesamiento).

3. **Ruta de Contingencia No-Code (`Nodo IF`):**
   * Valida la propiedad transcrita `{{ $json.text }}` mediante la condición `Is Not Empty`.
   * Desvía de manera segura ejecuciones con audios corruptos, silencias prolongadas o fallos de transcripción.

4. **Cerebro del Sistema (`AI Agent` + RAG):**
   * Mapea la transcripción textual de Whisper hacia el modelo central.
   * Conectado a la base de conocimiento vectorial previa (Módulo 5).
   * **Contención Financiera:** Incluye en el *System Prompt* la restricción explícita e imperativa de responder en un **máximo absoluto de 200 caracteres** para cuidar el consumo de tokens y cuotas de la API de voz.

5. **Voz del Agente (`ElevenLabs` - Action: *Text to Speech*):**
   * Mapea el output textual del agente (`{{ $json.output }}`).
   * Modelo: `Eleven Multilingual v2`.
   * Sliders de **Stability** y **Clarity** ajustados a la identidad de marca institucional.

6. **Salida Multimedia Binaria & Compliance:**
   * Devuelve el payload binario resultante vía `Respond to Webhook` (o Telegram).
   * **Compliance de Ciberseguridad:** Los archivos binarios de voz se procesan únicamente en memoria volátil durante la ejecución (`Save Data Processed Executions: Don't Save`) para resguardar la privacidad y cumplir normativas de protección de datos.

---

## 📂 Estructura de Archivos en el Repositorio

| Archivo | Descripción |
| :--- | :--- |
| `README.md` | Documentación técnica general del proyecto y arquitectura del workflow. |
| `AI_Automation_entrega_6.json` | Exportación completa del flujo principal de n8n para importar en la plataforma. |
| `Entrega_6__Javier_Velez.pdf` | Documento ejecutivo formal de la entrega, incluyendo capturas de pantalla de la configuración visual de cada nodo. |
| `audio-ejemplo.mp3` | Archivo de muestra de audio utilizado para la validación y pruebas de la transcripción en Whisper. |
| `velez-sub1-ent2.json` | Subflujo / componente de soporte del proyecto integrador (Módulo RAG / embeddings). |
| `velez-sub2-ent2.json` | Subflujo / componente de soporte del proyecto integrador (módulo de memoria / datos). |

---

## 🚀 Instrucciones de Uso e Importación

1. **Importar el Workflow en n8n:**
   * Abre tu instancia de n8n.
   * Haz clic en **Workflows** -> **Import from File...**
   * Selecciona el archivo `AI_Automation_entrega_6.json`.

2. **Configurar Credenciales:**
   * Configura las credenciales de **OpenAI API Key** en el nodo Whisper.
   * Configura tus credenciales de **ElevenLabs API Key** en el nodo de síntesis de voz.
   * Asocia tu modelo de lenguaje (Gemini / OpenAI) y base vectorial para el `AI Agent`.

3. **Ejecutar Pruebas:**
   * Activa el nodo `n8n Form Trigger` y abre la URL del formulario de prueba.
   * Adjunta el archivo `audio-ejemplo.mp3` incluido en este repositorio.
   * Haz clic en **Submit** y escucha el audio sintetizado generado por ElevenLabs como respuesta.

---

### 👨‍💻 Autor
**Javier Vélez**  

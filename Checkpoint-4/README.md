# 🌍 Sistema de Automatización de Atención al Cliente - Laguna Viajes
> **Curso:** AI Automation Avanzado  
> **Comisión:** #102005  
> **Pre-entrega:** 4  
> **Alumno:** Javier Vélez Ríos  

---

## 📌 Descripción del Proyecto

Este proyecto implementa una arquitectura basada en un **Worker Manager** impulsado por IA para la agencia de viajes premium *Laguna Viajes*. El sistema está diseñado para gestionar interacciones con clientes de manera inteligente, eficiente y segura. 

A través de integración con herramientas reales de negocio y autenticación **OAuth**, el flujo resuelve problemas clave de la automatización conversacional:
- 🛑 **Corta el bucle infinito** de respuestas automáticas.
- 🔍 Realiza **búsqueda previa (lookup)** antes de crear registros para evitar duplicados.
- ✍️ Genera borradores en **Gmail (Human-in-the-loop)** para revisión previa al envío.
- 🧹 Aplica un **set de limpieza de payloads** para asegurar datos estructurados.
- 📢 Notifica al equipo de ventas a través de **Slack**.
- 🧠 Hace uso de **memoria persistente a largo plazo** y validación de IDs de sesión (SSID).

---

## 🏗️ Arquitectura e Integraciones

El flujo combina orquestación determinista y capacidades predictivas de IA, estructurado en dos fases clave:

1. **Gestión de Sesión y Memoria:** Recepción del mensaje, validación de SSID, consulta e incorporación de memoria a largo plazo.
2. **Procesamiento e Integración:** Clasificación por IA Agent, limpieza de payload, enrutamiento a sub-workflows, creación de Draft en Gmail y notificación por Slack.

```
+------------------+     +-------------------+     +------------------+
|  Chat Trigger    | --> |  Validación SSID  | --> | AI Agent Manager |
|  (Entrada)       |     |  y Memoria        |     | (Clasificación)  |
+------------------+     +-------------------+     +------------------+
                                                            |
                                                            v
+------------------+     +-------------------+     +------------------+
| Slack Sales Group| <-- | Draft en Gmail    | <-- | Limpieza Payload |
| (Notificación)   |     | (Human-in-Loop)   |     | & Enrutamiento   |
+------------------+     +-------------------+     +------------------+
```

---

## 🔄 Flujo Detallado del Workflow

1. **Trigger Chat:** Inicio del evento por mensaje entrante del usuario.
2. **Validación de SSID:** Comprueba si es una nueva sesión o una conversación existente para cargar el contexto previo.
3. **AI Agent Clasificador:** Analiza el mensaje junto al historial previo recuperado.
4. **Set de Limpieza y Payloads:** Formatea y limpia la salida del modelo para garantizar integridad JSON.
5. **Enrutamiento Determinista:** Redirige la solicitud al subworkflow especializado según la clasificación.
6. **Creación de Draft en Gmail (Human-in-the-Loop):** Genera un borrador de correo listo para revisión humana previa.
7. **Actualización de Memoria:** Persiste los datos clave y el resumen consolidado a largo plazo.
8. **Notificación en Slack:** Envía una alerta con el estado del caso al canal del equipo de ventas.

---

## 🤖 Prompt del AI Agent (Orquestador Manager)

El agente opera con un prompt estructurado estrictamente para garantizar respuestas en formato JSON coherentes y predecibles.

```markdown
## ROL E IDENTIDAD
Eres el Orquestador Principal (Manager) de la agencia de viajes premium "Laguna Viajes".
Tu función es clasificar las solicitudes de los usuarios y sintetizar el estado de la interacción.

## ÁMBITO DE ACTUACIÓN
Tu área de acción se limita exclusivamente a analizar el mensaje entrante del usuario y
clasificarlo dentro de las categorías operativas de la agencia, considerando el
historial previo recuperado.

---

[INICIO DE CONTEXTO COMPARTIDO]
DATOS CLAVE HISTÓRICOS: {{ $json.fields.DATOS_CLAVE || 'Sin datos previos' }}
RESUMEN CONSOLIDADO: {{ $json.fields.RESUMEN_CONSOLIDADO || 'Primer contacto' }}
ESTADO DEL CASO: {{ $json.fields.ESTADO_DEL_CASO || 'Nuevo' }}
MENSAJE ENTRANTE DEL USUARIO: {{ $('Chat trigger').item.json.chatInput }}
[FIN DEL CONTEXTO COMPARTIDO]

---

## REGLAS DE CLASIFICACIÓN
Analiza la intención de la solicitud y asígnala a UNA de las siguientes categorías estrictas:

1. DESTINO: Solicitudes de información sobre destinos, tours específicos, disponibilidad de vuelos o itinerarios.
2. COTIZACION: Preguntas explícitas sobre precios, costos, presupuestos o tarifas.
3. OTROS: Cualquier otro tema, saludo, consulta general o mensaje fuera de los ámbitos anteriores.

---

## REQUISITOS OBLIGATORIOS DE SALIDA (ESTRICTO FORMATO JSON)
Responde EXCLUSIVAMENTE con un objeto JSON válido, sin bloques de markdown adicionales (sin ```json), sin explicaciones previa ni texto posterior.

Estructura requerida:

{
  "clasificacion": "DESTINO" | "COTIZACION" | "OTROS",
  "asunto_principal": "Breve descripción en una frase sobre la intención actual del cliente",
  "puntos_clave": [
    "Dato relevante 1 extraído de la charla",
    "Dato relevante 2 extraído de la charla"
  ],
  "accion_requerida": "Acción inmediata recomendada para el agente o sistema",
  "estado_del_caso": "Nuevo" | "En Proceso" | "Pendiente Cotización" | "Cerrado"
}
```

---

## 🖼️ Evidencias y Diagramas

### Flujo Principal - Primera Mitad
*Inserte aquí el diagrama o captura del inicio del workflow (Trigger, Memoria, AI Agent)*

### Flujo Principal - Segunda Mitad
*Inserte aquí el diagrama o captura del cierre del workflow (Enrutamiento, Gmail, Slack)*

### Integración y Notificación en Slack
*Inserte aquí la captura de pantalla de la evidencia de notificación enviada al canal de ventas de Slack*

---

## 🛠️ Tecnologías Utilizadas

- **Orquestación & Workflows:** n8n / Node-RED / Make (según plataforma elegida)
- **Modelos de IA:** LLM Orquestador / OpenAI / Anthropic
- **Integraciones:** Gmail API (OAuth 2.0), Slack API (OAuth 2.0)
- **Gestión de Memoria:** Base de datos relacional / Vector Store / Memoria Persistente de Sesión

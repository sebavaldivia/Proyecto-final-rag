# Proyecto Final - Agente Autónomo con RAG en n8n

## Caso: revisión de incidencias de asignación de proveedores de despacho

## 1. Resumen del caso de negocio

El proceso manual consiste en revisar solicitudes donde un pedido aparece registrado bajo un proveedor de despacho distinto del que informa el área operativa, consultar el registro del pedido, revisar el procedimiento de cada proveedor y redactar una respuesta o derivación. Se automatizan la lectura, comparación inicial, consulta del procedimiento y preparación de la propuesta; la corrección del dato y cualquier comunicación externa siguen bajo control humano.

### Volumen y ROI estimado

Datos de referencia proporcionados para el proceso: **15-20 minutos por solicitud** y **14-17 solicitudes por día**. Esto equivale a un esfuerzo manual aproximado de **210-340 minutos diarios (3,5-5,7 horas)**. La estimación previa de ahorro fue de **2-3 horas por día**, suponiendo que la automatización absorbe la lectura, búsqueda de reglas y redacción inicial, sin eliminar la revisión humana.

Con 22 días hábiles por mes: **44-66 horas recuperadas al mes**. La valorización monetaria depende del costo interno real por hora (C_h):

- Beneficio mensual bruto estimado = 44-66 × C_h.
- Ejemplo ilustrativo, no dato de nómina: si C_h = **$5.000 CLP/h**, el valor del tiempo recuperado sería **$220.000-$330.000 CLP/mes**.
- Supuesto de puesta en marcha: 10 horas de configuración × $5.000 = **$50.000 CLP** (costo de oportunidad único). Supuesto de mantenimiento: 2 horas × $5.000 = **$10.000 CLP/mes**.
- Beneficio neto antes de amortizar la puesta en marcha: **$210.000-$320.000 CLP/mes**. El retorno de la inversión inicial, bajo estos supuestos, sería aprox. **3,4-5,2 días hábiles**. Son valores de ejemplo; deben sustituirse por horas y costos reales.
- Fórmula de ROI mensual = ((beneficio valorizado - costo operativo mensual - costo de implementación amortizado) / (costo operativo mensual + costo de implementación amortizado)) × 100.

El ejemplo usa herramientas sin costo por token (Ollama local) y un nivel gratuito de Supabase/Google sujeto a cuotas. No incluye electricidad, hardware, conectividad ni tiempo real de mantenimiento; deben agregarse al cálculo local. El tiempo recuperado es capacidad operativa, no necesariamente ahorro contable directo.

### Por qué escala mejor y límites del ROI

El volumen repetitivo (14-17 casos/día) hace valioso estandarizar búsquedas y borradores. A medida que aumenta el número de casos, un mismo procedimiento RAG puede reutilizarse sin que una persona tenga que localizar manualmente las reglas en cada solicitud. La automatización deja de ser conveniente cuando el volumen cae tanto que el costo de mantener el sistema supera el tiempo recuperado, cuando la mayoría de los casos son excepciones que requieren intervención humana o cuando la tasa de errores/revisión consume el ahorro. El punto de equilibrio se calcula cuando `horas_recuperadas × C_h <= costos_operativos + implementación_amortizada + costo_de_revisión_adicional`.

## 2. Fuentes de entrada

El flujo combina dos fuentes distintas y un repositorio de conocimiento:

1. **Webhook (entrada de evento):** recibe la incidencia desde un formulario o sistema operativo. Campos sugeridos: `request_id`, `order_id`, `proveedor_reportado`, `detalle`, `solicitante`.
2. **Google Sheets (fuente operativa):** el nodo `Get row(s) in sheet` consulta la hoja `Registros_Despacho` utilizando el `order_id` exacto. Columnas sugeridas: `order_id`, `proveedor_registrado`, `estado_despacho`, `fecha_actualizacion`.
3. **Supabase Vector Store (RAG):** contiene procedimientos internos de validación y escalamiento. No debe contener datos personales o registros transaccionales si no son necesarios.

Ejemplo de payload para probar el Webhook (archivo `data/incidencia_webhook_ejemplo.json`):

```json
{
  "request_id": "INC-2026-001",
  "order_id": "PED-1002",
  "proveedor_reportado": "Rapido Norte",
  "detalle": "El área operativa informa que el pedido PED-1002 corresponde a Rapido Norte, pero en el registro aparece Carga Sur.",
  "solicitante": "operaciones"
}
```

## 3. Arquitectura del workflow principal

```mermaid
flowchart TD
  A[Webhook: incidencia] --> B[Code: validar y normalizar]
  B --> C[Google Sheets: Get row(s)]
  C --> D[Code: unir incidencia + registro]
  D --> E[AI Agent]
  M[Ollama Chat Model] --> E
  R[Supabase Vector Store: herramienta RAG] --> E
  V[Embeddings Ollama] --> R
  E --> F[Code: validar JSON / confianza]
  F --> G{IF: acción}
  G -->|RESPUESTA_PROPUESTA| H[Google Sheets: Bandeja_Revision]
  G -->|SOLICITAR_DATOS| I[Google Sheets: Datos_Pendientes]
  G -->|ESCALAR_HUMANO| J[Google Sheets: Escalamientos]
  H --> K[Respond to Webhook]
  I --> K
  J --> K
  X[Error Trigger, workflow separado] --> Y[Google Sheets: Errores]
```

### Función de cada nodo

| Nodo | Función / criterio |
|---|---|
| Webhook | Recibe una incidencia y actúa como primera fuente. |
| Code - validar y normalizar | Verifica campos mínimos y limpia espacios/capitalización sin cambiar el dato original. |
| Google Sheets - Get row(s) | Recupera el registro del pedido exacto; segunda fuente de datos. |
| Code - unir incidencia + registro | Combina el payload del Webhook con la fila encontrada y establece `registro_encontrado`. |
| AI Agent | Consulta el procedimiento RAG y decide una acción entre tres alternativas. |
| Ollama Chat Model | Ejecuta el modelo principal local. |
| Supabase Vector Store + Embeddings Ollama | Recupera los fragmentos más relevantes del procedimiento. |
| Code - validar JSON / confianza | Verifica el esquema, valores permitidos, confianza y revisión humana obligatoria. |
| IF encadenados | Derivan la acción a una de las tres bandejas de Google Sheets. |
| Google Sheets Append Row | Deja evidencia usable para revisión humana. No modifica el registro fuente. |
| Respond to Webhook | Devuelve al sistema solicitante la acción y el identificador de seguimiento. |
| Error Trigger (separado) | Registra ejecuciones fallidas de APIs/nodos y permite alertar al responsable. |

## 4. Por qué AI Agent + RAG y no una IA simple

El nodo **AI Agent** es necesario porque no solo redacta texto: decide qué herramienta consultar (procedimiento RAG), evalúa si hay datos suficientes y elige entre `RESPUESTA_PROPUESTA`, `SOLICITAR_DATOS` o `ESCALAR_HUMANO`. Una cadena simple podría redactar una respuesta secuencial, pero no está diseñada para elegir de forma autónoma entre herramientas. El RAG aporta reglas actualizables y evita depender solo del conocimiento general del modelo.

## 5. Decisión de modelo: local vs. nube

Se propone **Ollama local** para el agente principal, inicialmente con `qwen2.5:7b-instruct-q4_K_M` (o un modelo menor si el equipo no dispone de memoria suficiente). Para embeddings se propone `nomic-embed-text` (768 dimensiones), tanto al indexar como al consultar.

- **Privacidad:** los prompts y respuestas se procesan localmente en Ollama. Google Sheets y Supabase siguen siendo servicios externos; por eso se enviarán solo los datos necesarios y el vector store contendrá procedimientos generales, no información personal innecesaria.
- **Costo:** no hay cobro por token de Ollama; el costo se desplaza a hardware, energía y mantenimiento. Supabase y Google tienen cuotas y límites de sus planes gratuitos que deben verificarse antes de producción.
- **Independencia:** el sistema no depende de la disponibilidad o precio por token de un proveedor LLM cloud. Sí depende de la instancia n8n, el equipo local y el acceso a Google/Supabase.

## 6. RAG: ingesta, embeddings y recuperación

Documento inicial: `data/manual_proveedores_despacho.md`.

1. Preparar el modelo local: `ollama pull qwen2.5:7b` y `ollama pull nomic-embed-text`. Si el equipo es limitado, elegir una variante cuantizada o un modelo más pequeño.
2. Crear proyecto Supabase y ejecutar `sql/supabase_pgvector_setup.sql`. La tabla `documents` y la función `match_documents` usan `vector(768)`, coherente con `nomic-embed-text`.
3. En n8n crear la credencial Supabase y la credencial Ollama. Si n8n corre en Docker, `localhost` puede no apuntar al host; configurar la URL de Ollama según la red del contenedor.
4. Workflow de ingesta: Manual Trigger → lectura del documento (archivo TXT/MD) → Default Data Loader → Recursive Character Text Splitter (chunk inicial 800-1000; overlap 100-150) → Supabase Vector Store en modo Insert Documents; conectar Embeddings Ollama como entrada de embedding.
5. Workflow principal: Supabase Vector Store en modo Retrieve Documents (As Tool for AI Agent), conectado a la entrada `ai_tool` del AI Agent y a Embeddings Ollama. Nombre de herramienta `consultar_procedimientos`, descripción específica y límite inicial de 4-5 resultados.
6. Reindexar cuando el procedimiento cambie. Evitar duplicados: versionar por nombre/hash del documento y borrar/actualizar fragmentos previos antes de reinsertar.

El nodo Supabase Vector Store permite insertar documentos y conectarse directamente como herramienta al AI Agent; n8n documenta ambos patrones y el uso de `match_documents`.

## 7. System Prompt completo

El prompt completo está en `system_prompt.md`. Debe copiarse en el campo **System Message** del AI Agent. El agente solo tiene acceso de lectura al registro y al vector store. No tiene herramientas de modificación ni envío de mensajes.

## 8. Salida funcional

El resultado se guarda en Google Sheets, en una de estas hojas: `Bandeja_Revision`, `Datos_Pendientes` o `Escalamientos`. Columnas recomendadas: `request_id`, `order_id`, `proveedor_reportado`, `proveedor_registrado`, `accion`, `confianza`, `motivo`, `respuesta_propuesta`, `campos_faltantes`, `estado_revision`, `fecha`.

El estado siempre será `Pendiente de revisión humana`. El correo al proveedor o la corrección en el sistema fuente quedan fuera del alcance automático. En una fase posterior puede agregarse Gmail “Send and Wait for Approval” para una aprobación explícita, no un envío autónomo.

## 9. Manejo de errores y casos límite

| Riesgo | Control en el flujo | Resultado |
|---|---|---|
| Webhook incompleto o sin `order_id` | Code/IF valida campos obligatorios antes de consultar Sheets. | Responder 400 o registrar `SOLICITAR_DATOS`; no invocar al agente. |
| Google Sheets no encuentra pedido / error de Google | `Always Output Data` para tratar una búsqueda vacía; nodo con `On Error` hacia error output o Error Trigger para caídas. | `ESCALAR_HUMANO` o log de error; nunca inventar el registro. |
| Supabase no responde o RAG no recupera contexto | Error Trigger registra error; el prompt prohíbe contestar sin soporte documental. | Ejecución fallida controlada o escalamiento, sin enviar respuesta externa. |
| La IA devuelve JSON inválido o confianza baja | Code valida esquema, enumeraciones, confianza >= 0.80 para borradores y `requiere_revision_humana === true`. | `ESCALAR_HUMANO`; no se escribe un borrador como confirmado. |

Crear un workflow separado con `Error Trigger` y Google Sheets Append Row a la hoja `Errores`; asociarlo en Settings → Error Workflow del workflow principal. Además, usar un `IF` de validación antes de ejecutar cualquier acción que modifique datos. En este diseño no existe acción automática de modificación.




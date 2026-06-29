# 🏨 URBY — Agente Virtual del Hostal Urbano

Agente conversacional inteligente para la atención de huéspedes del Hostal Urbano, desarrollado con LangGraph bajo el patrón arquitectónico **ReAct** (Reasoning and Acting). El proyecto incluye búsqueda semántica, herramientas autónomas, guardrails de seguridad y un dashboard de observabilidad en tiempo real.

---

## Índice

- [Descripción general](#descripción-general)
- [Arquitectura](#arquitectura)
- [Herramientas del agente](#herramientas-del-agente)
- [Observabilidad](#observabilidad)
- [Seguridad y guardrails](#seguridad-y-guardrails)
- [Requisitos](#requisitos)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Ejecución](#ejecución)
- [Estructura del proyecto](#estructura-del-proyecto)
- [Autores](#autores)

---

## Descripción general

URBY es un asistente virtual diseñado para responder consultas de huéspedes sobre el Hostal Urbano. Opera sobre una base de conocimiento en PDF indexada con FAISS y puede derivar solicitudes a recepción cuando la consulta supera sus capacidades. Esta tercera fase añade una capa completa de observabilidad con métricas cuantitativas, trazabilidad de ejecución y visualización interactiva.

---

## Arquitectura

```
Usuario
   │
   ▼
Guardrails de seguridad
   │
   ▼
Agente ReAct (LangGraph + GPT-4o-mini)
   │
   ├── buscar_info_hostal        → FAISS (memoria de largo plazo)
   ├── consultar_disponibilidad  → lógica de reservas
   ├── registrar_derivacion      → CSV en Google Drive
   └── obtener_fecha_hora        → datetime
   │
   ▼
Sistema de observabilidad
   │
   ├── Logging → ejecucion_agente.log
   ├── Métricas → metricas_observabilidad.csv
   └── Dashboard → Gradio + Plotly
```

---

## Herramientas del agente

| Herramienta | Descripción |
|---|---|
| `buscar_info_hostal` | Búsqueda semántica en el índice FAISS (k=4 fragmentos). |
| `consultar_disponibilidad` | Gestiona solicitudes de disponibilidad de habitaciones. |
| `registrar_derivacion_recepcion` | Registra en CSV las consultas que requieren atención humana. |
| `obtener_fecha_hora` | Retorna la fecha y hora actual del sistema. |

---

## Observabilidad

Se implementaron **7 métricas** persistidas automáticamente en `metricas_observabilidad.csv`:

| Métrica | Descripción |
|---|---|
| `latencia_ms` | Tiempo de respuesta end-to-end por consulta. |
| `precision_score` | Score 0–1 basado en heurística de indicadores de falla. |
| `consistencia_score` | Ratio de turnos activos respecto al máximo del historial. |
| `exito / tipo_error` | Registro binario de éxito y tipo de falla. |
| `cpu_pct / mem_mb` | Uso de CPU y RAM del proceso (via `psutil`). |
| `pasos_react` | Cantidad de ciclos Razonar-Actuar-Observar por consulta. |
| `herramientas_usadas` | Lista de tools invocadas, para análisis de frecuencia. |

### Dashboard de observabilidad

Implementado con **Gradio + Plotly**. Incluye los siguientes paneles:

- **Latencia por consulta** — línea temporal con promedio y P95.
- **Precisión y consistencia** — evolución temporal con zona óptima marcada.
- **Frecuencia de uso de herramientas** — gráfico de barras.
- **Uso de recursos del sistema** — CPU (%) y memoria (MB) con doble eje.
- **Tasa de éxito vs error** — gráfico de dona.
- **Panel de KPIs** — resumen textual con recomendaciones automáticas.

### Detección de anomalías

| Patrón / Anomalía | Umbral |
|---|---|
| Alta latencia | > 2× promedio |
| Tasa de error elevada | > 15% |
| P95 crítico | > 8.000 ms |
| Alta tasa de derivación | Derivaciones > Búsquedas |
| Baja precisión | `precision_score` < 0.70 |

---

## Seguridad y guardrails

La función `evaluar_guardrails_seguridad()` filtra las consultas antes de procesarlas:

- **Datos personales sensibles** — RUT, tarjetas de crédito, correos y teléfonos.
- **Prompt injection** — patrones que intenten alterar el comportamiento del agente.

Cada evento bloqueado se registra con etiqueta `[SECURITY_BLOCK]` en el log, almacenando solo la longitud de la consulta (sin contenido sensible) y entregando al usuario un mensaje orientativo sin exponer los mecanismos internos.

---

## Requisitos

- Python 3.10+
- Google Colab (recomendado) o entorno con acceso a Google Drive
- Cuenta de OpenAI con acceso a `gpt-4o-mini` y `text-embedding-3-large`

---

## Instalación

```bash
pip install langchain langchain-openai langchain-text-splitters langchain-community \
            langgraph faiss-cpu gradio psutil plotly openpyxl pypdf
```

---

## Configuración

### Variables de entorno

| Variable | Descripción |
|---|---|
| `OPENAI_API_KEY` | Clave de acceso a la API de OpenAI (o GitHub Token si se usa GitHub Models). |
| `OPENAI_BASE_URL` | URL base de la API (opcional; solo si se usa un endpoint alternativo). |

### En Google Colab

Las variables se pueden configurar en **Secrets** de Colab con los nombres `GITHUB_TOKEN` y `GITHUB_BASE_URL`.

### Base de conocimiento

Coloca el archivo PDF del hostal en:

```
/content/drive/MyDrive/hostal/hostal_urbano_conocimiento.pdf
```

El índice FAISS se genera automáticamente y se guarda en:

```
/content/drive/MyDrive/hostal/hostal_faiss_index/
```

---

## Ejecución

Ejecuta las celdas del notebook `hostal_agente_fase3.ipynb` en orden:

| Celda | Descripción |
|---|---|
| 0 — Instalación | Instala dependencias. |
| 1 — Imports | Importa librerías. |
| 2 — Credenciales | Carga API keys e inicializa modelos. |
| 3 — Google Drive | Monta Drive y copia la base de conocimiento. |
| 4 — Vector Store | Indexa documentos en FAISS. |
| 5 — Herramientas | Define las 4 herramientas del agente y el sistema de trazabilidad. |
| 6 — Agente ReAct | Configura el system prompt y construye el agente con LangGraph. |
| 7 — Observabilidad | Implementa métricas, guardrails de seguridad, logging y la función principal `consultar_agente()`. |
| 8 — Limpieza de métricas | Elimina los CSV de métricas y seguridad para iniciar una nueva batería desde cero. **Ejecutar solo cuando se desee reiniciar el historial.** |
| 9 — Pruebas automáticas | Ejecuta una batería de 25 consultas (repetibles N rondas) para poblar los CSV con datos reales de observabilidad. Cubre información del hostal, servicios, reservas, consultas fuera de dominio, guardrails PII y prompt injection. |
| 10 — Interfaz Gradio | Lanza la interfaz conversacional del agente con visualización de latencia, herramientas y precisión por turno. |
| 11 — Loop interactivo | Modo consola. Escribe `salir` para terminar, `reset` para limpiar historial. |
| 12 — Dashboard | Lanza el panel de observabilidad con gráficos en tiempo real. |

---

## Estructura del proyecto

```
hostal_agente_fase3.ipynb       # Notebook principal
data/
└── hostal_urbano_conocimiento.pdf  # Base de conocimiento (copiada desde Drive)
```

Los siguientes archivos se generan automáticamente durante la ejecución y se persisten en Google Drive (`/content/drive/MyDrive/hostal/`):

```
hostal_faiss_index/             # Índice vectorial FAISS (generado en celda 4)
metricas_observabilidad.csv     # Métricas acumuladas por consulta (generado en celda 7)
eventos_seguridad.csv           # Registro de eventos bloqueados por guardrails (generado en celda 7)
derivaciones_recepcion.csv      # Registro de consultas derivadas a recepción (generado en celda 5)
```

El archivo de trazabilidad se genera localmente en el entorno de ejecución:

```
ejecucion_agente.log            # Log de trazabilidad con etiquetas estructuradas (generado en celda 5)
```

---

## Autores

- **José González**
- **Yaritxa González**

Asignatura: Ingeniería de Soluciones con IA — Sección 003D

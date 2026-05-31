# URBY — Agente Virtual del Hostal Urbano

> Sección 003D · José González — Yaritxa González

Agente conversacional basado en LangGraph + FAISS que atiende consultas de huéspedes, gestiona solicitudes de reserva y deriva casos a recepción de forma autónoma.

---

## Stack

| Componente | Tecnología |
|---|---|
| Agente | LangGraph `create_react_agent` |
| LLM | GPT-4o-mini (GitHub Models API) |
| Memoria largo plazo | FAISS + `text-embedding-3-small` |
| Memoria corto plazo | `chat_history` — ventana de 10 turnos |
| Interfaz | Gradio |
| Persistencia | Google Drive (índice FAISS + CSV derivaciones) |

---

## Requisitos

- Cuenta Google (Colab + Drive)
- Token de GitHub ([Settings → Developer settings → Personal access tokens](https://github.com/settings/tokens))

---

## Configuración

En **Colab → Tools → Secrets**, agrega:

| Secret | Valor |
|---|---|
| `GITHUB_TOKEN` | Tu token de GitHub |
| `GITHUB_BASE_URL` | `https://models.inference.ai.azure.com` |

Activa **Notebook access** en ambos.

---

## Archivo de conocimiento

1. Crea la carpeta `hostal` en Google Drive
2. Sube tu PDF ahí
3. Si el nombre es distinto al default, edita en **Celda 3**:

```python
RUTA_ARCHIVO_DRIVE = '/content/drive/MyDrive/hostal/TU_ARCHIVO.pdf'
```

---

## Ejecución

Corre las celdas en orden en Google Colab:

```
Celda 0  →  Instala dependencias
Celda 1  →  Imports
Celda 2  →  Credenciales + modelos
Celda 3  →  Monta Drive y copia el PDF        ← requiere autorizar Drive
Celda 4  →  Indexa FAISS                      ← ~60 seg la primera vez
Celda 5  →  Define herramientas
Celda 6  →  Construye agente
Celda 7  →  Inicializa memoria
Celda 8 →  Lanza Gradio                      ← genera enlace público
```

La Celda 9 ofrece un loop interactivo por consola (`salir` / `reset`).
La Celda 10 ejecuta las 6 pruebas funcionales automatizadas.

---

## Herramientas

| Herramienta | Se activa cuando... |
|---|---|
| `buscar_info_hostal` | Preguntas sobre servicios, precios, políticas o habitaciones |
| `consultar_disponibilidad` | El usuario menciona fechas o pide reserva |
| `registrar_derivacion_recepcion` | Solicitudes especiales fuera del manual |
| `obtener_hora_actual` | Consultas de check-in inmediato o dependientes del horario |

---

## Errores comunes

**FileNotFoundError** — El PDF no está en la ruta configurada. Ejecuta esto para encontrarlo:
```python
import os
for root, _, files in os.walk('/content/drive/MyDrive'):
    for f in files:
        if f.endswith(('.pdf', '.txt')): print(os.path.join(root, f))
```

**Error 429** — Límite diario alcanzado (`gpt-4o`: 50 req · `gpt-4o-mini`: 150 req). El sistema muestra un mensaje amigable automáticamente. Cambia a `gpt-4o-mini` en Celda 2 si usas `gpt-4o`.

**Gradio sin enlace** — El notebook se desconectó. Vuelve a ejecutar la Celda 8.

---

## Estructura

```
├── hostal_agente_fase2.ipynb
├── hostal_agente_fase2.py
├── informe_EP2_URBY.docx
└── README.md
```

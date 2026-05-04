# Sistema RAG para Consultas de Hostal Urbano

## Descripción

Este proyecto implementa un sistema de consultas inteligente para un hostal urbano utilizando un enfoque de **Retrieval-Augmented Generation (RAG)**.

El sistema permite responder preguntas de huéspedes basándose **exclusivamente en información interna del hostal**, evitando alucinaciones del modelo.

---

## Objetivo

* Automatizar respuestas a consultas frecuentes (horarios, servicios, políticas)
* Reducir carga operativa del personal
* Garantizar respuestas consistentes y trazables
* Evitar respuestas inventadas por el modelo

---

## Tecnologías Utilizadas

* **LangChain** → Orquestación del pipeline RAG
* **OpenAI (GPT-4o)** → Generación de respuestas
* **FAISS** → Base de datos vectorial en memoria
* **text-embedding-3-small** → Modelo de embeddings
* **Python** → Lenguaje base

---

## Arquitectura del Sistema

El sistema sigue un pipeline RAG clásico:

1. **Ingesta de datos**

   * Documentos del hostal definidos en una lista

2. **Chunking**

   * División de texto usando `RecursiveCharacterTextSplitter`
   * Parámetros:

     * `chunk_size=100`
     * `chunk_overlap=20`

3. **Embeddings**

   * Conversión de texto a vectores semánticos

4. **Indexación**

   * Almacenamiento en FAISS

5. **Recuperación**

   * Se obtienen los **3 fragmentos más relevantes (k=3)**

6. **Generación**

   * GPT-4o responde usando SOLO el contexto recuperado

---

## 💬 Prompt del Sistema

El modelo está restringido mediante un **system prompt**:

* Solo puede usar el contexto entregado
* No puede inventar información
* Debe indicar cuando no sabe la respuesta

Esto asegura comportamiento controlado y confiable.

---

## Cómo Ejecutar

### 1. Instalar dependencias

```bash
pip install -U langchain langchain-community langchain-openai langchain-text-splitters faiss-cpu
```

### 2. Configurar variables de entorno

En Google Colab:

```python
os.environ["OPENAI_API_KEY"] = userdata.get("GITHUB_TOKEN")
os.environ["OPENAI_BASE_URL"] = userdata.get("GITHUB_BASE_URL")
```

Nota: Se utiliza un proxy vía GitHub para acceder a la API.

---

### 3. Ejecutar el sistema

Simplemente corre el script y escribe preguntas en consola:

```text
Pregunta: ¿Cuál es el horario de check-in?
```

Para salir:

```text
salir
```

---

## 📊 Ejemplo de Uso

**Entrada:**

```
¿Se permiten mascotas?
```

**Salida:**

```
No se permiten mascotas dentro del hostal.
```

---

## Métricas

* Latencia promedio: **800 ms – 2400 ms**
* Recuperación: Top-K = 3 documentos
* Temperatura del modelo: **0.3** (respuestas consistentes)

---

## Limitaciones

* Base de conocimiento **estática (hardcoded)**
* No existe memoria conversacional
* No hay integración con sistemas reales (reservas, disponibilidad)
* Dependencia de conexión a API externa

---

## Mejoras Futuras

* Carga dinámica de documentos (PDF, BD, API)
* Implementar memoria conversacional
* Integración con sistema de reservas
* Uso de base vectorial persistente (Pinecone, Weaviate)
* Ajuste dinámico de `k` en el retriever

---

## Autor

Proyecto desarrollado por:

* José González
* Yaritxa González

Asignatura: **Ingeniería de Soluciones con IA**

---

## Licencia

Uso académico.

---

## Nota Técnica

El sistema incluye un ejemplo intencional de dato inconsistente:

> "se dispone de helipuerto para submarinos"

Esto permite evaluar si el modelo respeta el contexto incluso cuando contiene información absurda, demostrando el comportamiento del enfoque RAG.

---


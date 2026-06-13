# Podcast Generator con acento rioplatense

Generá tu propio podcast en español rioplatense a partir de un guion de texto y una pista musical de fondo. El pipeline sintetiza voz con **Chatterbox Multilingual TTS** (voces argentinas reales del dataset [ylacombe/google-argentinian-spanish](https://huggingface.co/datasets/ylacombe/google-argentinian-spanish)), aplica fonetización automática del yeísmo porteño y mezcla el audio final con tu música.

[![Abrir en Colab](https://colab.research.google.com/github/SabatoCom24212/podcast-tcom/blob/main/podcast_tcom.ipynb)]

---

## ¿Qué hace este proyecto?

- Sintetiza voz de dos locutores (**Clara** y **Leo**) con acento argentino usando referencias de voz reales.
- Preprocesa automáticamente el texto: aplica yeísmo rehilado (`ll`/`y` → `sh`) y pronunciación fonética de términos técnicos en inglés.
- Mezcla la voz con música de fondo: intro, cuerpo (voz sobre música atenuada) y outro con fade.
- Exporta el resultado como `podcast_final.mp3` listo para escuchar o publicar.
- Modo opcional: generá el diálogo automáticamente a partir de un texto usando la API de **Groq** (LLaMA 3.3).

---

## Requisitos

| Requisito | Detalle |
|---|---|
| GPU | NVIDIA con CUDA 12.4 (ej. T4 en Colab gratuito) |
| Python | 3.10 o superior |
| RAM GPU | ≥ 8 GB recomendado |
| Entorno | Google Colab (recomendado) o local con CUDA |

---

## Guía paso a paso

### Paso 1 — Abrí el notebook en Google Colab

Hacé clic en el botón **"Open in Colab"** del inicio de este README.

Una vez en Colab, verificá que el entorno de ejecución tenga GPU activada:

```
Menú → Entorno de ejecución → Cambiar tipo de entorno de ejecución → GPU (T4)
```

---

### Paso 2 — Prepará tu guion

Creá un archivo de texto llamado **`guion_dialogo.txt`** con el siguiente formato. Cada línea es una intervención de un locutor:

```
Clara: ¡Hola a todos! Bienvenidos a otro episodio del podcast.
Leo: Hoy vamos a hablar sobre inteligencia artificial y su impacto en la educación.
Clara: Exacto. Hay muchos cambios que ya se están viendo en las aulas.
Leo: Y lo más interesante es que recién estamos empezando.
Clara: Gracias por escucharnos, ¡hasta la próxima!
Leo: ¡Chau a todos!
```

**Reglas del formato:**
- Cada línea debe comenzar con `Clara:` o `Leo:` (el nombre del locutor, seguido de dos puntos).
- No uses asteriscos ni etiquetas HTML en el texto.
- El archivo debe estar guardado en **UTF-8**.

---

### Paso 3 — Conseguí tu música de fondo (opcional)

1. Entrá a [pixabay.com/es/music](https://pixabay.com/es/music/).
2. Buscá música libre de derechos (por ejemplo: "podcast background", "lo-fi", "corporate").
3. Descargá el archivo MP3.
4. Renombrá el archivo como **`musica.mp3`**.

> Si no subís música, el podcast se exportará solo con las voces.

---

### Paso 4 — Subí los archivos a Colab

En el panel izquierdo de Colab, hacé clic en el ícono de carpeta 📁 y luego en el ícono de subir archivo ⬆️. Subí:

- `guion_dialogo.txt`
- `musica.mp3` (si la tenés)

Ambos archivos deben quedar en el directorio raíz (`/content/`).

---

### Paso 5 — Configurá los parámetros (Celda de configuración global)

Buscá la sección `CONFIGURACIÓN GLOBAL` en el notebook y ajustá los valores según tu caso:

```python
GROQ_API_KEY    = "TU_API_KEY_AQUI"   # Solo si usás MODO_GUION = "groq"
ARCHIVO_GUION   = "guion_dialogo.txt"  # Nombre de tu archivo de guion
ARCHIVO_MUSICA  = "musica.mp3"         # Nombre de tu archivo de música

# "archivo" → usa el guion tal cual (recomendado)
# "groq"    → usa Groq para reescribir el guion como diálogo automáticamente
MODO_GUION      = "archivo"
```

**Modos disponibles:**

| Modo | Descripción |
|---|---|
| `"archivo"` | Usa el guion `.txt` exactamente como está. Tenés control total. |
| `"groq"` | Envía el contenido del `.txt` a Groq (LLaMA 3.3) y genera el diálogo automáticamente. Requiere una API key gratuita de [console.groq.com](https://console.groq.com). |

---

### Paso 6 — Ejecutá las celdas en orden

Ejecutá cada celda del notebook de arriba hacia abajo presionando `Shift + Enter` o usando `Menú → Entorno de ejecución → Ejecutar todo`.

**Descripción de cada celda:**

| # | Celda | Qué hace |
|---|---|---|
| 1 | Instalación de PyTorch con CUDA | Instala la versión compatible con la GPU de Colab. Tarda ~2 min. |
| 2 | Instalación de Chatterbox y dependencias | Instala el motor TTS y bibliotecas de audio. |
| 3 | Verificación de entorno | Confirma la versión de PyTorch y que CUDA esté disponible. |
| 4 | Reinstalación de `datasets` | Fija versiones compatibles para cargar el dataset de voces argentinas. |
| 5 | Test del dataset | Verifica que el dataset de voces argentinas sea accesible. |
| 6 | Pipeline principal | Carga el modelo, sintetiza la voz, mezcla con música y exporta el podcast. |

> La celda 6 es la más lenta: la primera vez que descarga el modelo TTS puede tardar **5–10 minutos**. Las ejecuciones siguientes en la misma sesión son más rápidas.

---

### Paso 7 — Escuchá y descargá tu podcast

Al terminar la celda 6, el notebook mostrará un reproductor de audio inline y creará el archivo `podcast_final.mp3` en `/content/`.

Para descargarlo:

```
Panel de archivos 📁 → clic derecho sobre podcast_final.mp3 → Descargar
```

---

## Parámetros avanzados de mezcla

Podés personalizar la estructura del podcast modificando estas constantes en la celda 6:

```python
INTRO_MS        = 4000   # Duración de la intro musical sola (ms)
FADE_ENTRADA_MS = 1500   # Duración del fade cuando entra la voz (ms)
VOL_FONDO_DB    = -26    # Volumen de la música de fondo en dB (más negativo = más suave)
OUTRO_MS        = 6000   # Duración del outro musical al final (ms)
FADE_OUTRO_MS   = 3000   # Duración del fade-in del outro (ms)
```

---

## Estructura del repositorio

```
├── podcast-tcom.ipynb       # Notebook principal
├── guion_dialogo.txt     # Tu guion (no incluido, debés crearlo)
├── musica.mp3            # Tu música de fondo (no incluida)
└── README.md
```

---

## Notas y limitaciones

- El modelo TTS requiere **GPU**. En CPU la síntesis es extremadamente lenta.
- Las voces de referencia se descargan del dataset de HuggingFace la primera vez; necesitás conexión a internet.
- Los IDs de locutor por defecto (`SPEAKER_ID_CLARA = 9697`, `SPEAKER_ID_LEO = 7508`) corresponden a hablantes argentinos del dataset. Podés cambiarlos para explorar otras voces.
- El preprocesador de texto aplica automáticamente yeísmo rehilado y pronunciación fonética de términos técnicos en inglés (ej. `software` → `sófwer`, `git` → `guit`).

---

## Dependencias principales

- [chatterbox](https://github.com/resemble-ai/chatterbox) — Motor TTS multilingüe
- [pydub](https://github.com/jiaaro/pydub) — Edición y mezcla de audio
- [groq](https://console.groq.com) — Generación de diálogos con LLaMA 3.3 (opcional)
- [datasets](https://huggingface.co/docs/datasets) — Acceso al dataset de voces argentinas
- [torchaudio](https://pytorch.org/audio) — Procesamiento de audio con PyTorch

---

## Licencia

MIT

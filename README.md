# EduDocs IA · Documentos escolares con IA (Gemini)

Aplicación de archivo único (`index.html`) para escuelas: sube documentos con **arrastrar y soltar**, guárdalos en la nube (Back4App), transcríbelos **íntegramente a Markdown con IA** y pregunta en lenguaje natural. La inteligencia artificial **Google Gemini** transcribe, analiza y responde.

**Sitio publicado:** https://argos4all.github.io/Ometecuhtli-Normatividad-IA/

## Características

- **Archivo único**: todo (HTML + CSS + JS) vive en `index.html`; sin instalación ni build.
- **IA Gemini** con cadena de respaldo entre modelos: si un modelo no está disponible, la app continúa automáticamente con el siguiente. Cada respuesta y transcripción indica el motor que la produjo (ej. "Gemini · gemini-3.5-flash-lite").
- **Transcripción integral al subir**: al subir un archivo, la IA lo transcribe de principio a fin siguiendo una regla de máxima precisión:
  - No omite párrafos, notas al pie, encabezados, números, fechas ni detalles.
  - Transporta las **tablas íntegramente a Markdown** conservando celdas, filas y columnas.
  - Respeta la **jerarquía original** de listas y apartados numerados.
  - No resume ni parafrasea; marca lo ilegible como `[texto no legible]`.
  - Ordena la salida según el orden exacto del documento original.
  - **Verifica y corrige los enlaces** del documento (paso adicional configurable).
  - El resultado se guarda como **archivo `.md` en Back4App** y queda disponible con el botón "Ver" y descarga directa.
- **PDF con texto y PDF escaneado**: extracción con pdf.js; si el PDF no tiene capa de texto, las páginas se rasterizan y se transcriben con visión (hasta 40 páginas por proceso).
- **Formatos**: PDF, DOCX, TXT, MD, CSV, JSON, XML y imágenes (máx. 25 MB por archivo); selección de carpeta completa.
- **Responsive**: PC, tablet y celular; tema **claro y oscuro** recordado entre visitas.
- **Indicadores de estado** en cabecera e inicio: Página, Back4App e IA Gemini (verificación automática cada 5 minutos y al iniciar).
- **Panel de administración** (contraseña): estado y configuración de Back4App, Gemini y GitHub Pages; claves enmascaradas; diagnóstico exportable; mantenimiento del esquema.
- **Chat con los documentos**: contexto seleccionable, respuestas en streaming y citas al documento de origen.
- **Análisis con IA**: resumen ejecutivo, puntos clave, análisis académico, cuestionario y mapa de ideas.
- **Modo embebido**: agrega `?embed=1` e insértala en cualquier sitio con un `<iframe>` (el código se genera en el panel Admin).

## Cómo se usa

1. Entra con el candado de la esquina (contraseña de administrador de la escuela).
2. Sube documentos en **Documentos** (arrastra y suelta, elige archivos o carpeta). La transcripción `.md` se genera sola, archivo por archivo, con barra de avance por fragmento.
3. Lee la transcripción con el botón **Ver** del documento, descárgala con **.md**, o genera resúmenes en **Análisis** y conversa en **Chat**.

Sin sesión de administrador la aplicación queda en modo consulta: cualquier visitante puede leer, analizar y preguntar, pero nadie puede subir ni eliminar documentos.

## Cadena de modelos Gemini

`gemini-3.5-flash-lite` y respaldos automáticos (`gemini-3.8-flash`, `gemini-3.5-flash`, `gemini-3.0-flash-lite`, `gemini-2.5-flash*`, `gemini-2.0-flash`). Si un modelo no responde o no existe en la cuenta, la app lo salta y continúa con el siguiente; el modelo en uso se muestra en el panel Admin.

Estados que la app muestra solos:

- **Clave rechazada o sin cuota** → píldora de IA en rojo/ámbar con la causa exacta en el panel Admin.
- **"User location is not supported"** → la red actual bloquea la API de Google; abre el sitio desde tu navegador y red habituales.
- **Fragmento con fallo puntual durante la transcripción** → el fragmento se marca en el `.md` y el proceso continúa con los demás.

## Backend y servicios

| Servicio | Uso |
| --- | --- |
| [Back4App](https://www.back4app.com) (Parse REST) | Guarda los documentos (clase `Documento`) y los archivos. Campos: `nombre`, `tipo`, `tamano`, `texto`, `archivo`, `estado`, `transcripcionMd`, `mdArchivo`, `motor`, `paginas`, `transcripcionFecha`, `subidoPor`. La app inicia sesión con un usuario de servicio para subir archivos si el servidor lo exige (error 130). |
| Google Gemini (`generativelanguage.googleapis.com`) | Análisis, chat y transcripción, con streaming (SSE) y cadena de respaldo entre modelos. |
| GitHub Pages | Hospeda el archivo `index.html` desde este repositorio. |

Las claves de servicio viven en el bloque `DEFAULTS` dentro de `index.html` (aplicación client-side). Recomendaciones: restringir la clave de Gemini al sitio publicado y rotarla periódicamente desde Google AI Studio; luego actualízala en Configuración sin tocar el código.

## Publicar una nueva versión

1. Edita o reemplaza `index.html`.
2. Haz *push* a la rama `main` (o usa "Add file → Upload files" en GitHub).
3. GitHub Pages despliega automáticamente en 1-2 minutos.

Las versiones anteriores se conservan en la carpeta `versiones/`.

## Estructura

```
index.html                      → aplicación completa (v3.3 · IA Gemini)
versiones/edudocs-ia-v2.0.html  → respaldo versión anterior (Gemini simple)
versiones/edudocs-ia-v1.0.html  → respaldo primera versión
```

## Notas y seguridad

- La contraseña de administrador se verifica localmente con hash SHA-256 y nunca se envía a ningún servidor; tras 5 intentos fallidos el acceso se bloquea 60 segundos.
- Los documentos se guardan en Back4App; el historial del chat vive en el navegador del usuario. La aplicación no usa analítica ni rastreo.
- El borrado de un documento elimina el registro; el archivo puede permanecer en el almacenamiento de Back4App (su borrado definitivo requiere la Master Key desde el dashboard).
- Si Gemini muestra "User location is not supported", la red actual bloquea la API de Google: abre el sitio desde tu navegador y red habituales y el sitio funciona con normalidad.

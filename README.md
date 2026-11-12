# EduDocs IA · Documentos escolares con IA

Aplicación de archivo único (`index.html`) para escuelas: sube documentos con **arrastrar y soltar**, guárdalos en la nube (Back4App), analízalos y pregunta con IA (Google Gemini).

**Sitio publicado:** https://argos4all.github.io/Ometecuhtli-Normatividad-IA/

## Características

- **Archivo único**: todo (HTML + CSS + JS) vive en `index.html`; sin instalación ni build.
- **Responsive**: adaptada a PC, tablet y celular; tema **claro y oscuro** con preferencia recordada.
- **Subida por arrastrar y soltar**: PDF, DOCX, TXT, MD, CSV, JSON o imágenes (máx. 25 MB); también selección de carpeta completa.
- **Extracción de texto automática** (pdf.js y mammoth.js vía CDN) para que la IA pueda leer los documentos.
- **Análisis con IA**: resumen ejecutivo, puntos clave, análisis académico, cuestionario y mapa de ideas.
- **Chat con los documentos**: preguntas en lenguaje natural con respuestas en streaming y citas al documento de origen.
- **Panel de administración** (contraseña): estado de Back4App, Gemini y GitHub Pages; configuración de modelo, temperatura y claves; mantenimiento del esquema.
- **Indicadores de estado** en la cabecera y en el inicio: Página, Back4App y IA Gemini.
- **Modo embebido**: agrega `?embed=1` e insértala en cualquier sitio con un `<iframe>` (el código se genera en el panel Admin).
- **Publicación sin tokens (PAT)**: la versión 2.0 no usa ni solicita Personal Access Tokens; el sitio se publica reemplazando `index.html` en el repositorio.

## Cómo se usa

1. Entra con el candado de la esquina (contraseña de administrador de la escuela).
2. Sube documentos en la sección **Documentos** (arrastra y suelta, elige archivos o carpeta).
3. Analiza en **Análisis** o conversa en **Chat**.

Sin sesión de administrador la aplicación queda en modo consulta: cualquier visitante puede analizar y preguntar, pero nadie puede subir ni eliminar documentos.

## Backend y servicios

| Servicio | Uso |
| --- | --- |
| [Back4App](https://www.back4app.com) (Parse REST) | Guarda los documentos (clase `Documento`: `nombre`, `tipo`, `tamano`, `texto`, `archivo`, `subidoPor`) y los archivos. La app inicia sesión automáticamente con un usuario de servicio para subir archivos si el servidor lo exige (error 130). |
| Google Gemini (`generativelanguage.googleapis.com`) | Análisis y chat, con streaming (SSE) y cadena de respaldo entre modelos si el configurado no está disponible. |
| GitHub Pages | Hospeda el archivo `index.html` desde este repositorio. |

Las claves de servicio viven en el bloque `DEFAULTS` dentro de `index.html` (aplicación client-side). Para producción se recomienda restringir la clave de Gemini al dominio `https://argos4all.github.io` y rotarla periódicamente.

## Publicar una nueva versión

1. Edita o reemplaza `index.html`.
2. Haz *push* a la rama `main` (o usa "Add file → Upload files" en GitHub).
3. GitHub Pages despliega automáticamente en 1-2 minutos.

La versión anterior se conserva en `versiones/edudocs-ia-v1.0.html`.

## Estructura

```
index.html                     → aplicación completa (v2.0)
versiones/edudocs-ia-v1.0.html → respaldo de la versión anterior
```

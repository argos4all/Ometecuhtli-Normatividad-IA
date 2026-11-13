# EduDocs IA · Documentos escolares con IA dual (Gemini + Kimi)

Aplicación de archivo único (`index.html`) para escuelas: sube documentos con **arrastrar y soltar**, guárdalos en la nube (Back4App), transcríbelos **íntegramente a Markdown con IA** y pregunta en lenguaje natural. Dos inteligencias artificiales — **Google Gemini** y **Kimi (Moonshot AI)** — trabajan de forma **complementaria**: si una no está disponible, la otra continúa automáticamente.

**Sitio publicado:** https://argos4all.github.io/Ometecuhtli-Normatividad-IA/

## Características

- **Archivo único**: todo (HTML + CSS + JS) vive en `index.html`; sin instalación ni build.
- **IA dual complementaria**: router interno que intenta Gemini y, ante cualquier fallo, continúa con Kimi (o viceversa según el modo elegido). Cada respuesta y transcripción indica el motor que la produjo (ej. "Gemini · gemini-3.5-flash-lite" o "Kimi · kimi-k3.1-pro").
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
- **Indicadores de estado** en cabecera e inicio: Página, Back4App, IA Gemini e IA Kimi (verificación automática cada 5 minutos y al iniciar).
- **Panel de administración** (contraseña): estado y configuración de Back4App, Gemini, Kimi y GitHub Pages; claves enmascaradas; diagnóstico exportable; mantenimiento del esquema.
- **Chat con los documentos**: contexto seleccionable, respuestas en streaming y citas al documento de origen.
- **Análisis con IA**: resumen ejecutivo, puntos clave, análisis académico, cuestionario y mapa de ideas.
- **Modo embebido**: agrega `?embed=1` e insértala en cualquier sitio con un `<iframe>` (el código se genera en el panel Admin).

## Cómo se usa

1. Entra con el candado de la esquina (contraseña de administrador de la escuela).
2. Sube documentos en **Documentos** (arrastra y suelta, elige archivos o carpeta). La transcripción `.md` se genera sola, archivo por archivo, con barra de avance por fragmento.
3. Lee la transcripción con el botón **Ver** del documento, descárgala con **.md**, o genera resúmenes en **Análisis** y conversa en **Chat**.

Sin sesión de administrador la aplicación queda en modo consulta: cualquier visitante puede leer, analizar y preguntar, pero nadie puede subir ni eliminar documentos.

## IA dual: cómo trabaja la complementariedad

| Modo | Comportamiento |
| --- | --- |
| **Auto (recomendado)** | Cada tarea intenta primero Gemini; si falla (clave, cuota, red, saldo), continúa Kimi automáticamente. En la transcripción larga, si Gemini cae a mitad del documento, Kimi termina los fragmentos restantes y el registro anota ambos motores ("Gemini + Kimi"). |
| **Solo Gemini** | Usa únicamente Gemini con su cadena de respaldo de modelos. |
| **Solo Kimi (Moonshot AI)** | Usa únicamente Moonshot con su cadena de respaldo de modelos. |

Cadena de modelos Moonshot: al iniciar, la app consulta el **catálogo de modelos** de la cuenta (`GET https://api.moonshot.ai/v1/models`) y arma la cadena automáticamente: primero los modelos de chat de la serie K3 disponibles (hoy `kimi-k3.1-pro` y luego `kimi-k3`, con contexto de hasta **1 millón de tokens** y modo de razonamiento integrado); los modelos de no-chat (visión, OCR, audio) se descartan. Si un modelo no está disponible, la app lo salta y continúa con el siguiente; cuando Moonshot publique nuevos modelos, la cadena se actualiza sola.

Cadena de modelos Gemini: `gemini-3.5-flash-lite` y respaldos (`gemini-3.8-flash`, `gemini-3.5-flash`, `gemini-3.0-flash-lite`, `gemini-2.5-flash*`, `gemini-2.0-flash`).

## Kimi desde el navegador (transporte directo)

Moonshot AI **permite CORS** desde el navegador, así que la app llama **directamente** a `https://api.moonshot.ai/v1/chat/completions` sin intermediarios (la clave solo funciona en el endpoint internacional `.ai`, no en `.cn`). Estados que la app muestra solos:

- **Cuenta sin saldo** → píldora en ámbar "Sin saldo en la cuenta · recarga créditos en platform.moonshot.ai" y Gemini cubre todas las tareas (modo complementario).
- **Red o firewall escolar bloquea Moonshot** → píldora en ámbar "Kimi no alcanzable desde esta red" y Gemini cubre todas las tareas.
- **Clave rechazada** → mensaje claro en el panel Admin con la causa exacta.

Opcional (redes escolares muy restrictivas): en Configuración hay un campo **"Kimi · Relay de CORS (opcional, respaldo)"**. Si tu red bloquea Moonshot, coloca ahí un worker propio que acepte el destino `api.moonshot.ai` en formato `https://worker.dev?url=…`; la app lo usará como transporte de respaldo. El worker actual de la escuela (`ometcuhtli.argos4all.workers.dev`) solo admite Z.ai y **no** es necesario para Kimi.

## Backend y servicios

| Servicio | Uso |
| --- | --- |
| [Back4App](https://www.back4app.com) (Parse REST) | Guarda los documentos (clase `Documento`) y los archivos. Campos: `nombre`, `tipo`, `tamano`, `texto`, `archivo`, `estado`, `transcripcionMd`, `mdArchivo`, `motor`, `paginas`, `transcripcionFecha`, `subidoPor`. La app inicia sesión con un usuario de servicio para subir archivos si el servidor lo exige (error 130). |
| Google Gemini (`generativelanguage.googleapis.com`) | Análisis, chat y transcripción, con streaming (SSE) y cadena de respaldo entre modelos. |
| Moonshot AI / Kimi (`api.moonshot.ai`) | **Kimi** para chat, análisis y transcripción, con descubrimiento automático de modelos y cadena de respaldo (`kimi-k3.1-pro` → `kimi-k3`); transporte directo (CORS permitido) y relay opcional. |
| GitHub Pages | Hospeda el archivo `index.html` desde este repositorio. |

Las claves de servicio viven en el bloque `DEFAULTS` dentro de `index.html` (aplicación client-side; la clave de Kimi va codificada en Base64 y se decodifica en runtime). Recomendaciones: restringir la clave de Gemini al sitio publicado y rotarla periódicamente; en Moonshot, revisa el consumo en platform.moonshot.ai y rota la clave si sospechas fuga.

## Publicar una nueva versión

1. Edita o reemplaza `index.html`.
2. Haz *push* a la rama `main` (o usa "Add file → Upload files" en GitHub).
3. GitHub Pages despliega automáticamente en 1-2 minutos.

Las versiones anteriores se conservan en la carpeta `versiones/`.

## Estructura

```
index.html                      → aplicación completa (v3.2 · IA dual Gemini + Kimi)
versiones/edudocs-ia-v2.0.html  → respaldo versión anterior (Gemini simple)
versiones/edudocs-ia-v1.0.html  → respaldo primera versión
```

## Notas y seguridad

- La contraseña de administrador se verifica localmente con hash SHA-256 y nunca se envía a ningún servidor; tras 5 intentos fallidos el acceso se bloquea 60 segundos.
- Los documentos se guardan en Back4App; el historial del chat vive en el navegador del usuario. La aplicación no usa analítica ni rastreo.
- El borrado de un documento elimina el registro; el archivo puede permanecer en el almacenamiento de Back4App (su borrado definitivo requiere la Master Key desde el dashboard).
- Si Gemini muestra "User location is not supported", la red actual bloquea la API de Google: la app sigue con Kimi automáticamente y el sitio funciona con normalidad desde el navegador y red habituales.

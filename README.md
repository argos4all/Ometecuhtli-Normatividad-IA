# EduDocs IA · Documentos escolares con IA dual (Gemini + GLM)

Aplicación de archivo único (`index.html`) para escuelas: sube documentos con **arrastrar y soltar**, guárdalos en la nube (Back4App), transcríbelos **íntegramente a Markdown con IA** y pregunta en lenguaje natural. Dos inteligencias artificiales — **Google Gemini** y **GLM (Z.ai)** — trabajan de forma **complementaria**: si una no está disponible, la otra continúa automáticamente.

**Sitio publicado:** https://argos4all.github.io/Ometecuhtli-Normatividad-IA/

## Características

- **Archivo único**: todo (HTML + CSS + JS) vive en `index.html`; sin instalación ni build.
- **IA dual complementaria**: router interno que intenta Gemini y, ante cualquier fallo, continúa con GLM (o viceversa según el modo elegido). Cada respuesta y transcripción indica el motor que la produjo (ej. "Gemini · gemini-3.5-flash-lite" o "GLM · glm-4.5-flash").
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
- **Indicadores de estado** en cabecera e inicio: Página, Back4App, IA Gemini e IA GLM (verificación automática cada 5 minutos y al iniciar).
- **Panel de administración** (contraseña): estado y configuración de Back4App, Gemini, GLM/Z.ai y GitHub Pages; claves enmascaradas; diagnóstico exportable; mantenimiento del esquema.
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
| **Auto (recomendado)** | Cada tarea intenta primero Gemini; si falla (clave, cuota, red, región), continúa GLM automáticamente. En la transcripción larga, si Gemini cae a mitad del documento, GLM termina los fragmentos restantes y el registro anota ambos motores ("Gemini + GLM"). |
| **Solo Gemini** | Usa únicamente Gemini con su cadena de respaldo de modelos. |
| **Solo GLM (Z.ai)** | Usa únicamente GLM con su cadena de respaldo de modelos. |

Cadena de modelos GLM: `glm-5.3-flash` (requiere saldo en Z.ai) → `glm-4.5-flash` (gratuito). Si la cuenta no tiene saldo para el modelo premium, la app lo detecta y usa el gratuito sin configurar nada; al recargar saldo, el premium se usa solo.

Cadena de modelos Gemini: `gemini-3.5-flash-lite` y respaldos (`gemini-3.8-flash`, `gemini-3.5-flash`, `gemini-3.0-flash-lite`, `gemini-2.5-flash*`, `gemini-2.0-flash`).

## Activar GLM desde el navegador (relay de CORS, opcional)

Z.ai limita las llamadas directas desde el navegador (su gateway no siempre responde el preflight CORS). La aplicación lo gestiona así:

1. Intenta la llamada directa a `https://api.z.ai` (funciona en la mayoría de los navegadores actuales).
2. Si el navegador la bloquea, usa el **relay opcional** configurado en **Admin → Configuración → "GLM · Relay de CORS"**.
3. Si no hay relay, muestra el estado "Bloqueada por CORS de Z.ai" y Gemini cubre todas las tareas.

Para publicar tu relay (gratis, ~2 minutos con cuenta de Cloudflare):

1. Entra a [workers.cloudflare.com](https://workers.cloudflare.com/) → "Create worker" y pega este código:

```js
export default {
  async fetch(req) {
    if (req.method === 'OPTIONS') return new Response(null, { headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Headers': '*',
      'Access-Control-Allow-Methods': 'POST, OPTIONS'
    }});
    const destino = new URL(req.url).searchParams.get('url');
    if (!destino || !destino.startsWith('https://api.z.ai/')) return new Response('url invalida', { status: 400 });
    const r = await fetch(destino, {
      method: 'POST',
      headers: { 'Content-Type': req.headers.get('content-type') || 'application/json', 'Authorization': req.headers.get('authorization') || '' },
      body: await req.text()
    });
    const resp = new Response(r.body, r);
    resp.headers.set('Access-Control-Allow-Origin', '*');
    return resp;
  }
};
```

2. Copia la URL del worker (ej. `https://mi-relay.tu-cuenta.workers.dev`) y pégala en **Admin → Configuración → GLM · Relay de CORS** → Guardar.
3. Pulsa "Probar" en la tarjeta GLM · Z.ai: la píldora debe quedar en verde.

> Consejo de seguridad: el relay solo reenvía la petición; la clave de Z.ai viaja desde el navegador del usuario. Puedes endurecer el worker restringiendo el `Origin` permitido a `https://argos4all.github.io`.

## Backend y servicios

| Servicio | Uso |
| --- | --- |
| [Back4App](https://www.back4app.com) (Parse REST) | Guarda los documentos (clase `Documento`) y los archivos. Campos: `nombre`, `tipo`, `tamano`, `texto`, `archivo`, `estado`, `transcripcionMd`, `mdArchivo`, `motor`, `paginas`, `transcripcionFecha`, `subidoPor`. La app inicia sesión con un usuario de servicio para subir archivos si el servidor lo exige (error 130). |
| Google Gemini (`generativelanguage.googleapis.com`) | Análisis, chat y transcripción, con streaming (SSE) y cadena de respaldo entre modelos. |
| Z.ai (`api.z.ai`) | GLM para chat, análisis y transcripción (cadena `glm-5.3-flash` → `glm-4.5-flash`), con respaldo por saldo y transporte directo/relay. |
| GitHub Pages | Hospeda el archivo `index.html` desde este repositorio. |

Las claves de servicio viven en el bloque `DEFAULTS` dentro de `index.html` (aplicación client-side). Recomendaciones: restringir la clave de Gemini al sitio publicado y rotarla periódicamente; recargar saldo en Z.ai para habilitar `glm-5.3-flash`.

## Publicar una nueva versión

1. Edita o reemplaza `index.html`.
2. Haz *push* a la rama `main` (o usa "Add file → Upload files" en GitHub).
3. GitHub Pages despliega automáticamente en 1-2 minutos.

Las versiones anteriores se conservan en la carpeta `versiones/`.

## Estructura

```
index.html                      → aplicación completa (v3.0)
versiones/edudocs-ia-v2.0.html  → respaldo versión anterior (Gemini simple)
versiones/edudocs-ia-v1.0.html  → respaldo primera versión
```

## Notas y seguridad

- La contraseña de administrador se verifica localmente con hash SHA-256 y nunca se envía a ningún servidor; tras 5 intentos fallidos el acceso se bloquea 60 segundos.
- Los documentos se guardan en Back4App; el historial del chat vive en el navegador del usuario. La aplicación no usa analítica ni rastreo.
- El borrado de un documento elimina el registro; el archivo puede permanecer en el almacenamiento de Back4App (su borrado definitivo requiere la Master Key desde el dashboard).
- Si Gemini muestra "User location is not supported", la red actual bloquea la API de Google: la app sigue con GLM automáticamente y el sitio funciona con normalidad desde el navegador y red habituales.

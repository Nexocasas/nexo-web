# Sitio Tomás Brinnand / Nexo Casas — guía para Claude

Este archivo lo lee Claude al empezar una conversación sobre este repositorio.
Resume cómo está armado el sitio y los criterios que se vienen usando. **Mantenelo al día** cuando cambie algo importante.

- **Dueño / interlocutor:** Matías (escribe en español rioplatense; respondé igual, claro y sin tecnicismos).
- **Publicado en:** https://www.tomasbrinnand.com.ar (GitHub Pages, rama `main`, dominio en `CNAME`).
- **Publicar = commit + push a `main`.** GitHub Pages tarda 2–5 min. Si Matías "no ve" un cambio, casi siempre es caché: pedirle Ctrl + Shift + R o ventana de incógnito.
- Sitio estático: HTML/CSS/JS sin build. No hay `package.json` ni dependencias.

## Mapa del sitio

| Archivo | Qué es |
|---|---|
| `index.html` | Página de **Tomás Brinnand** (carpintería): muebles, objetos, **Construcción** (tarjetas con fotos deslizables), historia, contacto. Los botones "Nexo Casas" llevan a `nexo/proyectos.html`. |
| `nexo/proyectos.html` | **Página principal de Nexo** (casi todo el trabajo pasa acá). Ver detalle abajo. |
| `panel.html` | **Panel privado de clientes** (contraseña). Lista clientes, link al formulario, bitácora, imprimir PDF, renombrar y eliminar. Datos en Supabase. |
| `nexo-formulario-cliente.html` | **Formulario del cliente** ("Tu Casa Nexo"): elige terminaciones por rubro. Link `?c=<token>`; con `&nexo=1` se abre en **modo Nexo** (agrega selector de "Estado" por ítem). |
| `nexo/informacion.html` | Página vieja "Qué incluye y cómo seguimos" (el contenido hoy vive dentro de las cotizaciones). |
| `nexo-cotizacion-v24.html`, `nexo/proyectos-respaldo.html` | Versiones viejas / respaldo. No editar salvo pedido. |
| `casas/`, `clientes/`, `muebles/`, `nexo-casas/`, etc. | Redirecciones de URLs cortas viejas. |
| `proyectos/<Casa X>/` | Fotos, videos y plano de cada proyecto. |
| `assets/planos/` | Planos de **Nuestros modelos** (`modelo-<m2>-<letra>.jpg`). |
| `assets/img/construccion/` | Fotos de las tarjetas de Construcción (página de Tomás). |
| `assets/img/cot/`, `assets/img/*_ref_*.jpg` | Imágenes del formulario del cliente. |
| `.github/workflows/supabase-keepalive.yml` | Tarea programada que evita que Supabase pause los proyectos (consulta de solo lectura cada 3 días). |
| `robots.txt` | Excluye `panel.html` y el formulario de buscadores. |

## `nexo/proyectos.html` — estructura

Orden de la página: encabezado con logo horizontal (isotipo + "Nexo / [ Taller de Casas ]") → franja **"Casas modulares hechas en la Patagonia"** como primera pantalla (con la foto `assets/img/nexo-hero-portada.jpg` de fondo) → **Nuestros Proyectos** (carrusel de tarjetas) → **Nuestros Modelos** (carrusel de tarjetas con opciones deslizables) → contacto/WhatsApp.

Todo el contenido está en arrays de JavaScript dentro del archivo (buscar por nombre):

- **`const proyectos = [ … ]`** — un objeto por proyecto. El **orden del array es el orden de las tarjetas**:
  ```js
  {
    nombre: 'Casa Lu',
    tag: 'PROYECTO 40M²',
    desc: 'Proyecto de 40m² …',            // los m² de desc deben coincidir con tag y plano
    heroFotos: [ BASE + 'Casa Lu/…jpg', … ], // [0] = portada del proyecto y de su tarjeta
    plano: { src: BASE + 'Casa Lu/planos.jpg',
             specs: [ {label:'Metros cuadrados', value:'40 m²'}, {label:'Módulos', value:'4'}, {label:'Ambientes', value:'2'} ] },
    secciones: [ { label: 'EXTERIOR', layout: 'duo'|'trio'|'trio-grid'|'solo', fotos: [ BASE + '…', … ] }, … ]
  }
  ```
  - `BASE = '../proyectos/'`. Las secciones con **más de 3** fotos se muestran como carrusel.
  - Se aceptan videos `.mp4` en `fotos`; cada `.mp4` necesita su portada `<mismo-nombre>.poster.jpg`.
  - Del plano **solo se muestran los m²** (arriba a la derecha, plano a todo el ancho); módulos/ambientes se guardan igual.
- **Menús de proyectos:** al agregar/reordenar proyectos hay que actualizar **tres** listas de botones `irAProyecto(n)` (`.nav-dropdown-item`, `.nav-mobile-sub`, `.menu-item`) para que el índice `n` coincida con el array.
- **`const descripciones = { 'archivo.jpg': 'texto' }`** — leyenda que aparece al abrir una foto en grande.
- **`const MODELOS = [ … ]`** — tarjetas de **Nuestros Modelos**. Cada modelo tiene `opciones` (plano por opción; `ambientes` por opción opcional) y `cotizacion`:
  ```js
  cotizacion: { id: '40', opcion: 'A', precio: 'U$S 72.000', ficha: { Superficie: '40 m²', Dormitorios: '1', Baños: '1' } }
  ```
- **`COT_WHATSAPP`** — número de WhatsApp (hoy +54 9 11 5908-1591, `5491159081591`). También aparece en el botón del cierre de la página y en `nexo/informacion.html`.

### Funcionalidades que ya existen (no romper)
- **Links directos a proyectos:** `#proyecto-<slug>` (ej. `#proyecto-casa-ampi-y-martin`). El botón "atrás" del navegador desde un proyecto vuelve a Nuestros Proyectos.
- **Cotizaciones privadas por link** (no aparecen en menús ni tarjetas): `#cotizacion-30`, `#cotizacion-40`, `#cotizacion-65`, `#cotizacion-90` (`#cotizacion-100` es alias de 90). Muestran un solo plano (`cotizacion.opcion`), precio, anticipo 5% calculado, forma de pago, WhatsApp y abajo "¿Qué incluye? / ¿Cómo seguimos?". La sección "Tu Casa Nexo" **solo** se ve dentro de una cotización.
- **Visor (lightbox):** recorre **toda la galería del proyecto** (todas las secciones), es **circular** (de la última vuelve a la primera), se desliza con el dedo, y también abre videos.

## Proyectos actuales (orden)
1. Casa Facundo · 40 m² · 2. Casa Lu · 40 m² · 3. Casa Luli · 35 m² · 4. Taller P & F · 30 m² · 5. Casa A & S · 60 m² · 6. Casa G & F · 35 m² · 7. Casa Marco · 40 m² · 8. Casa Ampi y Martín · 35 m² (carpeta `proyectos/Casa AyM/`)

## Modelos actuales
Casa 30 m² (opciones A, B; cotiza B, U$S 54.000) · Casa 40 m² (A, B, C; cotiza A, U$S 72.000) · Casa 65 m² (A, B; cotiza B, U$S 115.000) · Casa 85 – 90 m² (A, B; cotiza A como "Casa 90 m²", U$S 155.000).

## Cómo procesar material que manda Matías

Suele mandar fotos/videos por el chat (WhatsApp) o dejarlos en una carpeta (Descargas / OneDrive) si su computadora está conectada.

- **Fotos:** corregir orientación (EXIF), máx. 1600 px del lado mayor, JPG calidad ~84, progresivo. Nombres en minúscula con guiones (`aym-cocina-1.jpg`). Descartar duplicados casi idénticos y **cualquier imagen que no sea de la obra** (remitos, documentos con datos personales) avisando.
- **Planos (PDF):** rasterizar a 200 dpi (`pdftoppm -r 200`), recortar márgenes blancos (+60 px), ancho 1600 px, JPG ~85, guardar como `planos.jpg` en la carpeta del proyecto. Si viene vertical y Matías pide horizontal, rotar 90° antes de recortar.
- **Videos:** usar el **original sin recomprimir** (`ffmpeg -i in.mp4 -an -c:v copy -movflags +faststart out.mp4`) y generar la portada (`ffmpeg -ss 0.5 -i out.mp4 -frames:v 1 -q:v 2 out.poster.jpg`). Recomprimir los degradaba visiblemente.
- **"Eliminar foto N/M":** N/M es el contador del visor, que cuenta **todas** las fotos y videos del proyecto en orden de secciones. Calcular cuál es antes de borrar, y decirle a Matías cuál se sacó.
- Al agregar un proyecto: carpeta en `proyectos/`, objeto en `proyectos`, botones en los 3 menús, leyendas en `descripciones`, revisar que los m² coincidan en tag/desc/plano.

## Panel y formulario (Supabase)

- Proyecto Supabase **"Tu casa Nexo"** (`chplkjzxfqktftcefwsp`). Otro proyecto, **"Nexo-compras"** (`zjawwmwcjfttvcpchamn`), no lo usa ninguna página.
- Tabla `public.clientes` (`token`, `nombre`, `datos jsonb`, `creado`, `actualizado`).
- Funciones RPC: `cliente_get`, `cliente_save` (formulario, por token), `admin_listar`, `admin_crear`, `admin_renombrar`, `admin_eliminar` (panel, requieren contraseña).
- La contraseña del panel está **cifrada** (bcrypt) en `privado.admin_config`, verificada por `privado.admin_ok()`. **Nunca escribirla en el código ni en el repo.** Para cambiarla: `update privado.admin_config set pass_hash = extensions.crypt('<nueva>', extensions.gen_salt('bf',10)), actualizado = now() where id = 1;`
- "Opción base" en el formulario guarda la elección como `"Mantener opción cotizada"`; el panel y el PDF la muestran como "Opción base: <descripción>". Un ítem sin elección pero con estado "Definido" también se muestra como opción base.
- **Pedir confirmación a Matías antes de cualquier cambio que borre o modifique datos de clientes.**

## Criterios de trabajo

- Probar antes de publicar: servidor local (`python3 -m http.server`) + Playwright/Chromium (preinstalado) en escritorio (1440×900) y celular (390×844), sin errores de consola. Supabase y Google Fonts pueden estar bloqueados en el entorno: simular `supabase-js` con `page.route` para probar el panel/formulario.
- Commits en español, chicos y descriptivos. Hacer `git pull` antes de empezar.
- Mantener el estilo existente (fondo `#0d0b09`, dorado `--amber: #c9aa82`, tipografías `scale-variable` + DM Sans).
- Si un pedido es ambiguo (qué foto, qué opción, qué texto), preguntar una sola cosa breve; si es claro, hacerlo directo y contar el resultado.

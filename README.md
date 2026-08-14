# Scrolly Video Hero — Divi Code Module

Hero fullscreen cinematográfico controlado por scroll (scrubbing), listo para pegar en un **Code Module** de Divi. Sin plugins de animación, sin proceso de build.

Archivo del componente: [`scrolly-video-hero.html`](./scrolly-video-hero.html) — copia todo su contenido (bloque `<style>` + `<div>` + `<script>`) dentro de un único Code Module.

## Nota: espacios en blanco alrededor del Hero en Divi

Divi suele envolver los Code Modules en una fila (`.et_pb_row`) con ancho máximo y padding por defecto, y en una sección (`.et_pb_section`) con padding vertical (ej. 90px arriba/abajo). Eso provoca justo lo que se ve en capturas típicas: bordes blancos a los lados del video y un hueco debajo antes de la siguiente sección.

El componente ya lo resuelve por sí solo, sin que tengas que tocar nada en el builder de Divi:

- `.scrolly-video-hero` usa el truco clásico de *full-bleed* (`width:100vw` + márgenes negativos de `-50vw`) para romper el ancho máximo de la fila, sin necesidad de marcar "Make This Row Fullwidth".
- Una regla con `:has()` detecta la `.et_pb_section` / `.et_pb_row` / `.et_pb_column` / `.et_pb_module` que envuelve directamente al Hero y le pone `padding: 0; margin: 0;` **solo a esa** sección/fila — el resto del sitio no se ve afectado.

Si tu navegador de pruebas es muy antiguo y no soporta `:has()` (poco probable en 2026), como alternativa puedes poner manualmente el padding de esa Sección/Fila en Divi a `0px` en la pestaña Avanzado.

Nota adicional: el truco `100vw` puede añadir un pixelaje horizontal mínimo si la página tiene scrollbar vertical (comportamiento estándar del navegador, no un bug del componente). Si lo notas, puedes añadir `body { overflow-x: hidden; }` en Divi → Opciones del Tema → CSS personalizado.

## B. Configuración

Todo se controla desde el objeto `CONFIG` al inicio del `<script>`:

```js
var CONFIG = {
  videoUrl: "https://mi-sitio.com/video.mp4",
  scrollDistance: "250vh",
  showContent: true,
  showScrollIndicator: true,
  showProgress: true,
  scrub: 1,
  mobileEnabled: true
};
```

- **URL del video**: cambia `CONFIG.videoUrl`. Es el único lugar donde se define (`VIDEO_URL`); el script inyecta esa URL en el `<video>` antes de cargarlo.
- **Distancia de scroll**: cambia `CONFIG.scrollDistance` (`"150vh"`, `"200vh"`, `"300vh"`, `"400vh"`...). Determina cuánto debe scrollear el usuario para recorrer el video completo mientras el Hero permanece pineado.
- **Activar/desactivar texto**: pon `CONFIG.showContent = false` para eliminar título, subtítulo y CTA del DOM (el video sigue funcionando igual). También puedes ocultarlo solo visualmente quitando la clase `hero-content-enabled` del `<div class="scrolly-video-hero__content">` en el HTML.
- **Activar/desactivar indicador de scroll**: `CONFIG.showScrollIndicator = false`.
- **Activar/desactivar barra de progreso**: `CONFIG.showProgress = false`.
- **Overlay**: cambia la variable CSS `--svh-overlay-color` en `.scrolly-video-hero` (por defecto `rgba(0,0,0,0.20)`).
- **Viñeta**: variable CSS `--svh-vignette-color`.
- **Colores de texto / acento / progreso**: variables CSS `--svh-text-color`, `--svh-accent-color`, `--svh-progress-track`, `--svh-progress-fill`.
- **Título / subtítulo / CTA**: edita directamente el texto dentro de `.scrolly-video-hero__title`, `.scrolly-video-hero__subtitle` y `.scrolly-video-hero__cta` en el HTML (y el `href` del CTA).
- **Suavidad del scrub**: `CONFIG.scrub` (número = segundos de "elasticidad"; `true` = sin retraso).
- **Comportamiento en móvil**: `CONFIG.mobileEnabled = false` fuerza la versión estática (sin scrubbing) en pantallas ≤767px.

## C. Optimización del video (exportación MP4)

Recomendado para un balance premium/rendimiento:

| Parámetro | Recomendación |
|---|---|
| Resolución | 1920×1080 (Full HD). Evita 4K: no aporta calidad perceptible en Hero y dispara el peso. |
| FPS | 24–30 fps (no uses 60 fps, no aporta nada al scrubbing y duplica peso) |
| Códec | H.264 (`libx264`), perfil High, contenedor `.mp4` — máxima compatibilidad |
| Bitrate | ~4–6 Mbps para 1080p (VBR, 2 pasadas) |
| Keyframes | Intervalo corto: 1 keyframe cada 15–24 frames (`-g 15` a `-g 24` en ffmpeg). Esto es crítico: el scrubbing hace *seeks* constantes, y con pocos keyframes el `currentTime` salta o tarda en decodificar. |
| Audio | Ninguno (elimina la pista de audio, el video va `muted` siempre) |
| Duración | 8–15 segundos suele ser el punto dulce para este efecto |
| Peso recomendado | Idealmente 3–8 MB total. Si supera ~12 MB, considera recortar duración o bajar bitrate |

Ejemplo de comando ffmpeg:

```bash
ffmpeg -i input.mov -an -c:v libx264 -profile:v high -pix_fmt yuv420p \
  -vf "scale=1920:-2" -r 25 -g 25 -b:v 5M -maxrate 6M -bufsize 8M \
  -movflags +faststart output.mp4
```

`-movflags +faststart` es importante: mueve los metadatos al inicio del archivo para que `loadedmetadata` se dispare cuanto antes.

## D. Checklist de pruebas

- [ ] **Desktop**: scroll hacia abajo avanza el video de forma fluida y proporcional
- [ ] **Desktop**: scroll hacia arriba retrocede el video exactamente al contrario
- [ ] **Desktop**: el Hero queda pineado durante todo `SCROLL_DISTANCE` y se libera limpiamente al terminar (sin saltos ni espacios en blanco)
- [ ] **Tablet** (768–1024px): mismo comportamiento, sin overflow horizontal
- [ ] **Móvil** (≤767px): scrubbing fluido si `mobileEnabled: true`; si se desactiva, se muestra Hero estático sin bloquear el scroll
- [ ] **Chrome**: sin errores en consola, `ScrollTrigger.refresh()` funciona tras resize
- [ ] **Firefox**: mismo comportamiento de scrubbing y pin
- [ ] **Safari / iOS Safari**: seeking del video funciona sin pantallas negras ni congelamientos; si el navegador no permite seeking preciso, no debe romperse el Hero
- [ ] **Resize de ventana**: tras cambiar el tamaño, `ScrollTrigger.refresh()` recalcula correctamente el pin y la distancia de scroll
- [ ] **Cambio de orientación en móvil**: el Hero se reajusta sin quedar roto ni con scroll bloqueado
- [ ] **`prefers-reduced-motion: reduce` activado**: se muestra el primer frame de forma estática, sin pin ni scrubbing, y la navegación continúa con normalidad
- [ ] **Video que falla al cargar** (URL rota): el Hero muestra el fondo de respaldo elegante, sin pantalla negra ni errores visibles
- [ ] **Doble inicialización**: si el Code Module se ejecuta dos veces (p. ej. por AJAX de Divi), el Hero no se inicializa dos veces (`data-scroll-video-initialized="true"`)
- [ ] **Dos Hero en la misma página** (si aplica): GSAP/ScrollTrigger se cargan una sola vez, cada instancia funciona de forma independiente

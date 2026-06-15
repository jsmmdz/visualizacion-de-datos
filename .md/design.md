# Sistema de Diseño — Visor 3D de Incidentes UAECOB

> **Nota de honestidad:** este documento describe el sistema **ya presente** en
> `incidentes-3d/index.html`; no es una especificación aspiracional. Se redacta
> porque el archivo original `desing.md.txt` estaba vacío (0 bytes). Los puntos
> marcados **TBD** no están decididos. El `desing.md.txt` (con typo) puede borrarse.

---

## 1. Tokens (fuente: `:root` en index.html)

| Token | Valor | Semántica |
|---|---|---|
| `--ink` | `#f2f2f2` | Texto primario |
| `--ink-dim` | `#8a8a8a` | Texto secundario · **A11y:** roza el límite AA en tamaños ≤9px (p.ej. `.dp-label`); considerar `~#9a9a9a` |
| `--bg-panel` | `rgba(10,10,12,0.55)` | Superficie flotante translúcida · **fallback `0.92`** sin `backdrop-filter` |
| `--line` | `rgba(255,255,255,0.14)` | Bordes |
| `--c-sinsignos` | `#b3122e` | Acento categoría SIN SIGNOS |
| `--c-heridos` | `#ff6a00` | Acento HERIDOS |
| `--c-rescatados` | `#00ffb3` | Acento RESCATADOS |
| `--c-afectados` | `#ffe000` | Acento AFECTADOS |
| `--c-expuestos` | `#2979ff` | Acento EXPUESTOS |
| `--font` | Inter / system-ui | UI |
| `--mono` | JetBrains Mono | Datos y números (`tabular-nums` en métricas) |

**Estética:** dark premium, paneles flotantes modulares translúcidos (`backdrop-filter:
blur(12px) saturate(150%)`), basemap monocromático B/N con acentos vibrantes
reservados **solo** a datos WebGL y acentos de UI de datos.

---

## 2. Paneles modulares

`#timeline`, `#tooltip`, `#detail-panel` comparten el patrón: `background: var(--bg-panel)`
+ `backdrop-filter` + `1px solid var(--line)`. El fondo es translúcido para dejar ver
el lienzo WebGL del mapa en el `body`. Fallback `@supports not (backdrop-filter)` sube
la superficie a `0.92` opaco para conservar contraste AA donde no hay blur.

---

## 3. Catálogo de movimiento (con comportamiento bajo `prefers-reduced-motion`)

| Animación | Parámetros | Reduced-motion |
|---|---|---|
| Muelle del panel | resorte `k=205, d=24, m=1` (sub-amortiguado, micro-overshoot) | salto directo al destino |
| Motion-blur | curva `V0=2.2 / V1=15 / GAMMA=2.4 / MAX=7` (zona muerta + potencia) | filtro **off** |
| Inercia de paneo | `F=0.92`, umbrales en px (`START=2, STOP=0.3, MAX=50`) | **sin glide** |
| Count-up métricas | `easeOutCubic`, `dur = clamp(420 + n·22, 420, 900)ms`, `n≤2` salto directo | valor final directo |
| Barras (relleno) | `transform: scaleX()` (compositor), mismo reloj que el número, piso `0.06` | scaleX final directo |
| Cascada de entrada | `transform:translateY(8px→0)+opacity`, stagger `55ms`, `cubic-bezier(0.16,1,0.3,1)` | instantánea |
| Tilt 3D decorativos | radio `150px`, lerp `0.12` seguimiento / `0.05` retorno, máx `±7°/±5°`, scale `1.03`, idle sin/cos `2.5px/0.4°` | estático (no se instala) |

**Reglas de rendimiento (objetivo duro: 60fps del mapa WebGL):**
- En cualquier bucle por-frame, **solo** `transform` y `opacity` (compositor). Nunca
  `width/top/left/box-shadow` (layout/paint).
- **Cero `requestAnimationFrame` nuevo permanente:** el tilt + idle viven dentro del
  `map.on("render")` ya always-on (por `triggerRepaint`). El único rAF nuevo es el
  count-up: temporal y autoterminante (<1s, solo al abrir la tarjeta).
- El reveal de métricas se **difiere** hasta `moveend` (flyTo terminado) **y**
  `blurOn===false`, para no solapar con el `feGaussianBlur` de pantalla completa.
- `will-change` **transitorio** (set en `openPanel`, clear en el reposo del muelle),
  nunca permanente en el panel de 380px.
- `contain: layout paint` en `#dp-metrics` aísla el reflow del count-up.

---

## 4. Hover 3D no-clickeable (decorativos)

Los decorativos (`header h1`, `#phase-indicator`, `#legend`) llevan `pointer-events:none`
para **no bloquear el mapa**. Como `pointer-events:none` anula también `:hover` en CSS,
el "hover 3D" se calcula por **proximidad del cursor** (radio 150px) en JS, desde un
único `window` `pointermove` global (no desde `map.on("mousemove")`, que se congela sobre
los paneles). Honra la física de `interactions.md` (repulsión 150px, retorno lerp 0.05,
flotación idle sin/cos), reubicándola sobre sustratos ya presupuestados.

**Decisión sobre el "enjambre de partículas" de `interactions.md`:** NO se añade un 2º
canvas — competiría por fill-rate de GPU con el WebGL de 16k instancias y con el
motion-blur full-screen, arriesgando los 60fps. Su intención (vida reactiva al cursor)
se honra vía el proximity-tilt. *Opción avanzada (TBD):* modular el uniform `uLens` del
shader existente para repulsión de partículas en GPU sin draw-calls nuevos.

---

## 5. Deuda técnica declarada

- **Acentos duplicados:** los 5 hex de categoría están en CSS (`--c-*`) **y** en JS
  (`COLORS`, `new THREE.Color(0x…)`). Doble fuente de verdad ⇒ riesgo de deriva.
  **Recomendación:** declarar `:root` como fuente única y leer en JS con
  `getComputedStyle(document.documentElement).getPropertyValue('--c-*')` al boot.

---

## 6. Accesibilidad (parcial — TBD donde no decidido)

- ✅ `prefers-reduced-motion`: interruptor global sobre las 7 animaciones.
- ✅ `:focus-visible` en controles (`#play-btn`, `#close-panel`, slider).
- ✅ Fallback de contraste sin `backdrop-filter`.
- **TBD:** focus-trap del diálogo `#detail-panel`, región `aria-live` para el canvas
  opaco, revisión de contraste de `--ink-dim` en tamaños pequeños.

# Incidentes UAECOB 2020 — Visualización 3D

Visualización espacio-temporal en 3D de los incidentes atendidos por el Cuerpo Oficial de Bomberos de Bogotá (UAECOB), enero–agosto 2020 (datos abiertos, 20.348 registros).

## Vistas

| Vista | Archivo | Descripción |
|---|---|---|
| Mapa interactivo | `incidentes-3d/index.html` | MapLibre GL + Three.js. Heatmap macro, lente de contexto 3D al hover y edificios procedimentales instanciados (cajas con ventanas emisivas) en las coordenadas geográficas de cada caso. Al hacer click se abre la tarjeta de datos y el edificio del incidente se enciende con fuego + humo. Timeline con decaimiento temporal. |

## Ejecución

```bash
cd "visualizacion de datos"   # raíz del repo
python -m http.server 8801 --directory incidentes-3d
```

Abrir:
- http://localhost:8801/ (mapa interactivo)

Requiere conexión a internet (CDN de MapLibre/Three.js y teselas del mapa base).

## Codificación visual (mapeo de datos)

- **Categoría dominante** (prioridad SIN SIGNOS → HERIDOS → RESCATADOS → AFECTADOS → EXPUESTOS) → color del edificio y del fuego: rojo carmesí, naranja, verde-cian, amarillo, azul eléctrico
- **Severidad** ponderada `5·sinSignos + 3·heridos + 2·rescatados + afectados + expuestos` → volumen y altura del edificio + nº de partículas de fuego/humo
- **Víctimas** (sin signos / heridos) → ventanas de las plantas altas en llamas (parpadeo procedimental)
- **EXPUESTOS** → disco glow aditivo en la base del caso seleccionado (grosor ∝ conteo)
- **Estrato** → halo en la base, radio/intensidad inversamente proporcionales `(7−e)/6`
- **Fecha** → opacidad con decaimiento exponencial `exp(−0.025·Δdías)` controlada por timeline
- **Fuego + humo** → emisores sobre la cubierta (w×d) del edificio seleccionado; la llama se adapta a la huella del edificio

Sin assets 3D externos: toda la volumetría es procedimental (cajas + shaders GLSL) e instanciada (un solo draw call para los ~16K edificios).

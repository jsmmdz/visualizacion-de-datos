# Incidentes UAECOB 2020 — Visualización 3D

Visualización espacio-temporal en 3D de los incidentes atendidos por el Cuerpo Oficial de Bomberos de Bogotá (UAECOB), enero–agosto 2020 (datos abiertos, 20.348 registros).

## Vistas

| Vista | Archivo | Descripción |
|---|---|---|
| Mapa interactivo | `incidentes-3d/index.html` | MapLibre GL + Three.js. Heatmap macro, lente de contexto 3D al hover, micro-análisis con partículas por categoría de afectación y timeline con decaimiento temporal. |
| Escena cinemática | `incidentes-3d/cinematic.html` | Ciudad isométrica nocturna 100% procedimental: edificios en llamas, anillos concéntricos, bloom, profundidad de campo y aberración cromática. Datos del 19 de agosto de 2020 ± 10 días. |

## Ejecución

```bash
cd "visualizacion de datos"   # raíz del repo
python -m http.server 8801 --directory incidentes-3d
```

Abrir:
- http://localhost:8801/ (mapa interactivo)
- http://localhost:8801/cinematic.html (escena cinemática)

Requiere conexión a internet (CDN de MapLibre/Three.js y teselas del mapa base).

## Codificación visual (mapeo de datos)

- **SIN SIGNOS** → rojo carmesí, columna de humo ascendente (volumen ∝ conteo)
- **HERIDOS** → naranja, pulso estroboscópico (frecuencia ∝ conteo)
- **RESCATADOS** → verde-cian, ondas concéntricas en expansión (radio ∝ conteo)
- **AFECTADOS** → amarillo, chispas con movimiento browniano (densidad ∝ conteo)
- **EXPUESTOS** → azul eléctrico, contorno glow estático (grosor ∝ conteo)
- **Estrato** → halo en la base, radio/intensidad inversamente proporcionales `(7−e)/6`
- **Fecha** → opacidad con decaimiento exponencial `exp(−0.025·Δdías)` controlada por timeline
- **Servicio/Causa** → forma primitiva: pirámide (fuego), esfera (rescate), cilindro (otros)

Sin assets 3D externos: toda la volumetría es procedimental (primitivas + shaders GLSL).

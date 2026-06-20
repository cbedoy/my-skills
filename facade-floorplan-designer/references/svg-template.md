# SVG template — visualize:show_widget

Base template y clases disponibles para todos los SVG generados con `visualize:show_widget` en este skill.

## Template base

```svg
<svg viewBox="0 0 680 H" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="10" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M 0 0 L 10 5 L 0 10 Z" fill="var(--t)"/>
    </marker>
  </defs>
  <style>
    .t  { font: 11px monospace; fill: var(--t); }
    .ts { font: 9px monospace; fill: var(--t); opacity: 0.7; }
    .th { font: 11px monospace; fill: var(--t); font-weight: bold; }
    .arr { stroke: var(--t); fill: none; stroke-width: 0.5; }
    .c-gray  { fill: #d4d4d4; stroke: none; }
    .c-coral { fill: #e8a090; stroke: none; }
    .c-amber { fill: #d4a050; stroke: none; }
    .c-white { fill: #f0f0f0; stroke: none; }
  </style>
  <!-- contenido aquí -->
</svg>
```

## viewBox

- Ancho fijo: `680`
- Alto variable: calcular como `(fondo_m × escala) + 140` en plantas, o suficiente para la volumetría + acotación en fachadas.
- Área segura de dibujo: `x=40` a `x=640` (~600px útiles de ancho).

## Clases de texto

| Clase | Tamaño | Uso |
|---|---|---|
| `t` | 11px, normal | Etiquetas generales, títulos |
| `ts` | 9px, opacidad 0.7 | Notas, áreas, cotas, leyendas |
| `th` | 11px, bold | Nombres de espacios, encabezados |

Todas usan `fill: var(--t)` para adaptarse al tema claro/oscuro.

## Clases de color (zonas de material en fachada)

| Clase | Color | Material sugerido |
|---|---|---|
| `c-gray` | `#d4d4d4` | Block/concreto aparente o aplanado claro |
| `c-coral` | `#e8a090` | Ladrillo o tabique aparente |
| `c-amber` | `#d4a050` | Madera (acentos, cancelería, pérgolas) |
| `c-white` | `#f0f0f0` | Aplanado blanco o pintura clara |

Los colores se usan con `class="c-gray"` en rects/paths, no con `fill` inline.

## Elementos compartidos

- **Flecha de norte**: `<g transform="translate(600,60)">...` (definida en `floorplan-drawing-guide.md`)
- **Escala gráfica**: barra horizontal con marcas, cerca de la leyenda
- **Acotación**: líneas con clase `ts`, marcas en extremos con línea corta perpendicular
- **Flecha `#arrow`**: se usa con `class="arr"` y `marker-end="url(#arrow)"` para escaleras y ejes

## Reglas

- No usar `fill` inline cuando exista una clase `c-*` para el propósito.
- No poner texto dentro de `defs` o `style`.
- El `marker` del arrow se define una vez en `defs` y se reusa.
- Si se necesita un color no cubierto por la paleta, declararlo con nombre semántico (ej. `c-earth`) y agregarlo al `<style>` con un comentario breve de su uso.

---
name: facade-floorplan-designer
description: >-
  Use this skill when the user wants to design a house or building facade and floor plans for a personal construction project — describing room programs (planta baja, planta alta, terraza), construction type, lot dimensions, or architectural style and wanting Claude to generate a conceptual facade elevation and floor plan layout. Triggers on mentions of "diseñar fachada", "planos de mi casa", "distribución de planta baja/alta", "concepto arquitectónico", "boceto de fachada", "render de fachada", "layout de terreno", or when the user describes room-by-room space needs (recámaras, sala, cocina, terraza, cochera) wanting a visual floor plan and elevation. Also use for early-stage house design exploration — massing, window-to-wall proportions, material choices, climate-responsive design. This produces conceptual schematics for design exploration, not construction-grade or permit-ready drawings.
---

# Diseñador de fachadas y plantas arquitectónicas

Actúa como arquitecto conceptual ayudando a explorar y visualizar ideas para una casa propia, antes de pasar a un proyecto ejecutivo con un arquitecto/DRO certificado. El objetivo es traducir una descripción en lenguaje natural (qué espacios quiere, cuántos niveles, qué estilo) en una fachada y plantas esquemáticas con proporciones razonables — no un plano para trámite o construcción.

## Límite importante (decirlo una vez, al entregar el primer set de planos)

Estos planos y fachadas son conceptuales: ayudan a explorar distribución, proporciones y volumetría. No sustituyen un levantamiento topográfico, un proyecto ejecutivo, ni la revisión de un Director Responsable de Obra (DRO) o arquitecto certificado — necesarios para trámites de uso de suelo, licencia de construcción y cumplimiento del Reglamento de Construcción local. Decir esto una sola vez por conversación, no repetirlo en cada iteración.

## Step 1 — Levantar el programa arquitectónico

Si el usuario no dio esta información, preguntar en un solo mensaje (o inferir de lo ya compartido en la conversación):

**Terreno:**
- Dimensiones del lote (frente × fondo) o m² totales — si no las tiene, pedir un estimado razonable
- Orientación del frente (norte/sur/oriente/poniente) — necesaria para las recomendaciones de Step 5
- Ciudad/clima (default: Aguascalientes si no se especifica — clima semi-árido, ver `references/climate-design-principles.md`)

**Programa por nivel** (preguntar solo los niveles que aplican):
- Planta baja: qué espacios (sala, comedor, cocina, recámara en PB, baño, cochera, etc.) y si los quiere abiertos/integrados o separados
- Planta alta: recámaras, baños, estudio, balcón
- Terraza/azotea: uso (asador, alberca, lavandería, jardín)

**Estilo y construcción:**
- Estilo deseado (contemporáneo, minimalista, colonial, industrial, mediterráneo, etc. — ver `references/architectural-styles.md`) — si no lo sabe, ofrecer 2-3 referencias para que elija
- Tipo de construcción (block y losa tradicional, prefabricada, muros de carga) — afecta proporciones de vanos en la fachada

No avanzar a generar planos hasta tener al menos: niveles a incluir, lista de espacios principales por nivel, y estilo aproximado. El resto (m² exactos, materiales) se puede asumir con un rango razonable, declarando el supuesto explícitamente.

## Step 2 — Calcular áreas y proporciones

Antes de dibujar, calcular y mostrar una tabla de áreas. Usar rangos de referencia estándar (declarar que son rangos típicos, no medidas exactas del usuario):

| Espacio | m² típico |
|---|---|
| Recámara principal | 12-16 |
| Recámara secundaria | 9-12 |
| Cocina | 8-12 |
| Sala-comedor integrado | 18-25 |
| Baño completo | 4-5 |
| Medio baño | 2-3 |
| Cochera (1 auto / 2 autos) | ~14 / ~28 |

Mostrar: m² de construcción por nivel, m² de construcción total, y % de ocupación del suelo (huella de PB / m² del terreno). Mencionar que los reglamentos municipales suelen limitar el COS (coeficiente de ocupación del suelo) alrededor de 70-80%, y que el dato exacto debe verificarse con el municipio correspondiente — no inventar el porcentaje normativo.

Si el programa pedido no cabe razonablemente en el terreno dado, decirlo aquí con el cálculo, antes de dibujar nada.

## Step 3 — Generar planta(s): baja, alta, terraza

Usar `visualize:show_widget` (una llamada por nivel, SVG). Seguir `references/floorplan-drawing-guide.md` para escala, muros, puertas, ventanas, acotación y etiquetas — son convenciones de dibujo arquitectónico simplificado, no los iconos genéricos de diagramas de flujo.

Escribir un párrafo breve de texto entre cada planta explicando qué resuelve ese nivel y cómo se conecta con el anterior (ubicación de la escalera, por ejemplo). No apilar las llamadas a la herramienta sin texto entre ellas.

## Step 4 — Generar la fachada

Usar `visualize:show_widget` para el alzado frontal, siguiendo `references/facade-elevation-guide.md`: volumetría por nivel, proporción de vanos (ventanas/puertas) respecto a muro, tipo de cubierta (plana/inclinada), y zonas de material indicadas por color con una leyenda corta. Es un alzado esquemático de líneas y volúmenes — no fotorrealista, y hay que dejarlo claro si el usuario pregunta por un "render".

La fachada debe ser consistente con las plantas ya generadas: mismo ancho de frente, mismo número de niveles, y ventanas ubicadas donde hay espacios habitables (no donde hay baños o muros de escalera ciegos).

## Step 5 — Recomendaciones de diseño

Cerrar con recomendaciones breves basadas en principios verificables, no en gusto estético — ver `references/climate-design-principles.md`:

- **Orientación solar**: con la orientación dada, qué fachada recibe más sol poniente (la más crítica en clima cálido-seco) y si conviene reducir vanos ahí o agregar parteluces/aleros
- **Relación vano-muro**: % de ventana recomendado por fachada según orientación
- **Materiales**: 2-3 opciones con razón técnica (inercia térmica, mantenimiento, costo relativo) — no solo estética
- **Ventilación cruzada**: si la distribución de plantas la permite, y por qué

Separar siempre lo que es principio de diseño verificable (proporción, orientación solar, física básica de transferencia de calor) de lo que es preferencia estética — no presentar gustos como reglas.

## Step 6 — Iterar

El usuario va a querer ajustar (mover una recámara, cambiar el techo, agrandar la cochera). Para cambios menores, regenerar solo el nivel o la fachada afectada, no todo el set. Si el cambio afecta el ancho de frente o el número de niveles, regenerar también la fachada para mantener consistencia.

## Reglas

- Nunca presentar el resultado como plano ejecutivo o apto para trámite — siempre como "concepto" o "boceto de distribución".
- Si faltan datos críticos (orientación, niveles, dimensiones del lote), preguntar antes de dibujar — no asumir en silencio datos que cambiarían la solución.
- Para datos normativos específicos (COS, CUS, alturas máximas, restricciones de fraccionamiento), indicar que deben verificarse con el Reglamento de Construcción del municipio — nunca inventar un número normativo.
- Las recomendaciones bioclimáticas se basan en principios generales de diseño pasivo, no en una simulación específica del sitio; para afinar (ángulos solares exactos, ganancia térmica calculada) se requiere un estudio bioclimático con datos del predio.

## Referencias

- `references/svg-template.md` — Template base y clases compartidas para todos los SVG (viewBox, tipografía, paleta de colores, marcadores)
- `references/architectural-styles.md` — Descripción operativa de estilos arquitectónicos (contemporáneo, minimalista, colonial, industrial, mediterráneo, rústico) para guiar la elección del usuario y la generación de fachadas
- `references/floorplan-drawing-guide.md` — Convenciones para dibujar plantas arquitectónicas en SVG (escala, muros, puertas, ventanas, acotación, etiquetas)
- `references/facade-elevation-guide.md` — Convenciones para dibujar alzados/fachadas esquemáticos (proporción, niveles, vanos, cubiertas, zonas de material)
- `references/climate-design-principles.md` — Principios de diseño bioclimático para clima semi-árido (Aguascalientes y similares): orientación, % de vano por fachada, sombreado, inercia térmica

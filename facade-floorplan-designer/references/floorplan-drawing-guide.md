# Guía de dibujo: plantas arquitectónicas en SVG

Estas son convenciones específicas para dibujar una planta arquitectónica (a escala, con muros, vanos y acotación) usando `visualize:show_widget`. Se apoya en el sistema de diseño general (viewBox 680px, clases `t`/`ts`/`th`, paleta `c-*`) pero con reglas propias de planta, distintas a un diagrama de flujo o estructural genérico.

## 1. Definir la escala antes de dibujar

El área segura del viewBox es de ~600px de ancho (x=40 a x=640) y la altura es flexible.

1. Tomar la dimensión más larga del nivel a dibujar (frente o fondo, en metros).
2. `escala_px_por_metro = 580 / dimension_mas_larga_m` (deja margen para acotación).
3. Redondear la escala a un número limpio si es posible (10, 15, 20, 25, 30, 40 px/m) para que las cotas se vean limpias.
4. Aplicar la misma escala a las dos dimensiones (frente y fondo) — nunca distorsionar proporciones.
5. Si el lote es muy alargado (ej. 6m × 25m), considera dibujar el nivel rotado 90° (fondo en horizontal) para aprovechar mejor el ancho de 680px, y dejarlo claro con la flecha de norte.

Calcular `viewBox height = (fondo_m × escala) + 140` (deja espacio para acotaciones inferiores, escala gráfica y leyenda).

## 2. Elementos y cómo dibujarlos

**Muros** — rectángulos delgados, no líneas sueltas. Grosor real ~15cm → `grosor_px = 0.15 × escala` (mínimo 3px para que se vea). Usar `fill="var(--t)" opacity="0.85"` para que funcione en ambos modos, sin necesidad de clase `c-*`.

```svg
<rect x="100" y="100" width="6" height="240" fill="var(--t)" opacity="0.85"/>
```

**Puertas** — un vano (corte en el muro) más un arco de 90° mostrando el barrido de la puerta. El arco usa un `<path>` con `fill="none"`:

```svg
<!-- vano de 90cm en un muro vertical -->
<path d="M 100 180 A 27 27 0 0 1 127 207" fill="none" stroke="var(--t)" stroke-width="1" opacity="0.6"/>
<line x1="100" y1="180" x2="100" y2="207" stroke="var(--t)" stroke-width="1.5"/>
```

El radio del arco = ancho del vano en px (puerta estándar 90cm → `0.9 × escala`).

**Ventanas** — vano sin arco, marcado con dos líneas paralelas delgadas cruzando el hueco del muro (en vez del relleno sólido del muro):

```svg
<line x1="300" y1="98" x2="380" y2="98" stroke="var(--t)" stroke-width="1"/>
<line x1="300" y1="103" x2="380" y2="103" stroke="var(--t)" stroke-width="1"/>
```

**Habitaciones** — el contorno lo dan los muros, no se dibuja un rect de fondo (deja el interior transparente). Etiqueta centrada en el espacio: nombre (`class="th"`) + área aproximada (`class="ts"`), dos líneas con `tspan`:

```svg
<text x="220" y="200" text-anchor="middle" dominant-baseline="central">
  <tspan class="th" x="220" dy="0">Cocina</tspan>
  <tspan class="ts" x="220" dy="16">9.5 m²</tspan>
</text>
```

**Escalera** (si aplica) — serie de líneas paralelas cortas dentro de un rectángulo, con una flecha indicando sentido de subida y la etiqueta "sube"/"baja":

```svg
<g stroke="var(--t)" stroke-width="0.5" opacity="0.7">
  <line x1="500" y1="100" x2="540" y2="100"/>
  <line x1="500" y1="118" x2="540" y2="118"/>
  <line x1="500" y1="136" x2="540" y2="136"/>
  <line x1="500" y1="154" x2="540" y2="154"/>
</g>
<line x1="520" y1="154" x2="520" y2="105" class="arr" marker-end="url(#arrow)" opacity="0.7"/>
<text class="ts" x="520" y="170" text-anchor="middle">sube</text>
```

**Cochera** — contorno punteado (no es un cuarto cerrado) con el ícono opcional de un rectángulo sugiriendo el auto:

```svg
<rect x="40" y="300" width="160" height="280" rx="2" fill="none" stroke="var(--t)" stroke-width="0.5" stroke-dasharray="4 4" opacity="0.6"/>
```

## 3. Acotación (dimension lines)

Una línea de cota por lado relevante del nivel (frente y fondo como mínimo), fuera del contorno del muro:

```svg
<!-- línea de cota con marcas en los extremos -->
<line x1="40" y1="60" x2="40" y2="56" stroke="var(--t)" stroke-width="1"/>
<line x1="640" y1="60" x2="640" y2="56" stroke="var(--t)" stroke-width="1"/>
<line x1="40" y1="58" x2="640" y2="58" stroke="var(--t)" stroke-width="0.5"/>
<text class="ts" x="340" y="48" text-anchor="middle">12.00 m</text>
```

No acotar cada habitación individualmente — basta el perímetro general y, si es relevante, 1-2 cotas internas clave (ej. ancho de cochera). El detalle de áreas ya va en la etiqueta de cada cuarto (paso anterior).

## 4. Norte y escala gráfica

Incluir siempre, en una esquina libre:

```svg
<g transform="translate(600,60)">
  <line x1="0" y1="20" x2="0" y2="0" class="arr" marker-end="url(#arrow)"/>
  <text class="ts" x="0" y="34" text-anchor="middle">N</text>
</g>
```

Y una escala gráfica simple cerca de la leyenda (barra con marcas cada metro, etiqueta "1 m"):

```svg
<line x1="40" y1="H-30" x2="<40+escala>" y2="H-30" stroke="var(--t)" stroke-width="1"/>
<text class="ts" x="40" y="H-16">1 m</text>
```//(sustituir H y el valor de escala calculado)

## 5. Checklist antes de finalizar cada planta

1. ¿La escala es la misma en X y Y? (nunca estirar el dibujo para que "quepa")
2. ¿Cada habitación tiene nombre + m² aproximado, y el texto cabe dentro del espacio dibujado?
3. ¿Las puertas tienen arco de barrido y las ventanas son líneas dobles — se distinguen a simple vista?
4. ¿Hay al menos una cota de frente y una de fondo, y una flecha de norte?
5. ¿El viewBox height cubre todo el dibujo + 40px de margen inferior?
6. Si hay planta alta, ¿el contorno exterior coincide en ancho con la planta baja? (no debe "flotar" más ancha o más angosta sin razón estructural explicada en el texto)

# Guía de dibujo: alzados/fachadas esquemáticos en SVG

Convenciones para dibujar el alzado frontal (fachada) de la casa con `visualize:show_widget`. Es un alzado de líneas y volúmenes a proporción — no un render fotorrealista. Debe ser consistente con las plantas generadas en el mismo set (mismo ancho de frente, mismo número de niveles).

## 1. Escala y línea de tierra

Usar la misma escala (px/metro) calculada para las plantas, si el ancho de frente es el mismo — así la fachada se ve proporcional a las plantas del mismo set.

- Línea de tierra (nivel de piso terminado) cerca de la parte inferior del viewBox, con margen para acotación debajo.
- Altura típica de entrepiso: 2.7-3.0 m por nivel (declarar el supuesto si no se especificó).
- Apilar los niveles desde la línea de tierra hacia arriba: PB, luego PA si existe, luego pretil/parapeto de terraza si aplica.

```svg
<line x1="40" y1="420" x2="640" y2="420" stroke="var(--t)" stroke-width="1"/>
<text class="ts" x="20" y="424">±0.00</text>
```

## 2. Volumen del edificio

Contorno general como un solo `<path>` o `<rect>` por nivel — no hay necesidad de mostrar el muro con grosor aquí (eso es de planta). El alzado es una silueta:

```svg
<rect x="100" y="120" width="480" height="300" fill="none" stroke="var(--t)" stroke-width="1"/>
```

Si hay un volumen que sobresale (balcón, cochera con techo distinto, doble altura), dibujarlo como un rectángulo adicional que se conecta al contorno principal — sin que las líneas se crucen sin sentido.

## 3. Cubierta

**Losa plana** (más común en construcción tradicional mexicana): una línea horizontal simple cerrando el volumen arriba, con un pretil (parapeto) de 80-100cm si se quiere mostrar uso de azotea/terraza:

```svg
<rect x="100" y="100" width="480" height="20" fill="none" stroke="var(--t)" stroke-width="1"/>
```

**Cubierta inclinada**: un triángulo o trapecio sobre el volumen, con la pendiente dibujada a un ángulo razonable (no exagerado, 20-30° visualmente):

```svg
<path d="M 90 120 L 340 60 L 590 120 Z" fill="none" stroke="var(--t)" stroke-width="1"/>
```

## 4. Vanos (puertas y ventanas) — coherencia con la planta

Esta es la parte que más se rompe si no se hace con cuidado: cada vano del alzado debe corresponder a un espacio real de la planta de ese nivel, en la posición horizontal aproximada que le corresponde según el ancho del frente.

1. Mirar la planta del nivel: ¿qué espacios dan al frente (la fachada que se está dibujando)?
2. Espacios habitables (sala, recámanas, cocina si da al frente) → ventana proporcional al tamaño del cuarto, no minúscula. Ancho típico 1.2-2.0 m, alto 1.2-1.5 m, antepecho (parte baja) a ~0.9-1.0 m del piso de ese nivel.
3. Baños o vestidores → ventana pequeña y alta (0.4-0.6 m de ancho, antepecho alto ~1.8-2.0m) o ninguna si da a otro lado.
4. Acceso principal → puerta de 0.9-1.1 m de ancho, 2.1 m de alto, ubicada donde la planta muestra la entrada.
5. Cochera → vano grande (2.5-5.5 m según 1 o 2 autos) a nivel de piso.

```svg
<!-- ventana recámara -->
<rect x="150" y="180" width="90" height="80" fill="none" stroke="var(--t)" stroke-width="1"/>
<line x1="195" y1="180" x2="195" y2="260" stroke="var(--t)" stroke-width="0.5"/>
```

No inventar vanos "para que se vea bien" sin verificar contra la planta — si no se ha generado la planta de ese nivel todavía, decirlo y dibujar el alzado como volumetría sin vanos detallados, o generar primero la planta.

## 5. Proporción vano-muro (referencia visual rápida)

Usar como guía de proporción al dibujar (el detalle climático está en `climate-design-principles.md`):

| Orientación de la fachada | % de vano sobre el muro (referencia) |
|---|---|
| Norte | 30-40% |
| Sur | 25-35% (con alero) |
| Oriente | 20-30% |
| Poniente | 15-20% |

Si la fachada que se dibuja da al poniente y el usuario pidió ventanales grandes ahí, mencionarlo como advertencia técnica en el texto (no negarse a dibujarlo, solo señalar el costo energético).

## 6. Zonas de material (color como indicador, no textura)

No hay forma de mostrar textura de material real en este sistema — usar color plano de la paleta como código, con leyenda explícita debajo del SVG (en el texto de la respuesta, no dentro del SVG):

- `c-gray` (claro) → block/concreto aparente o aplanado claro
- `c-coral` → ladrillo o tabique aparente
- `c-amber` → madera (acentos, cancelería, pérgolas)
- Mantener máximo 2 colores de material por fachada — más de eso se ve como collage, no como propuesta de materiales coherente.

```svg
<rect x="100" y="220" width="200" height="200" class="c-gray"/>
```

Aclarar siempre en el texto: "los colores indican zonas de material sugerido, no el render final — útil para decidir combinaciones antes de cotizar con el constructor."

## 7. Checklist antes de finalizar la fachada

1. ¿El ancho total coincide con el ancho de frente usado en las plantas del mismo set?
2. ¿El número de niveles dibujados coincide con los niveles que el usuario pidió?
3. ¿Cada vano corresponde a un espacio real de la planta correspondiente, en la posición horizontal correcta?
4. ¿La proporción vano-muro por orientación es razonable o se señaló la advertencia si no lo es?
5. ¿Hay máximo 2 colores de material, con leyenda en el texto de la respuesta (no dentro del SVG)?
6. ¿La línea de tierra y la acotación de altura total están presentes?

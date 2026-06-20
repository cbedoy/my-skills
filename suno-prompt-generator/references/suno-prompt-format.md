# Formato de prompt para Suno — referencia rápida

## Estructura del style string

```
[género principal], [subgénero/fusión], [2-4 elementos de instrumentación], [mood], [tempo BPM], [instrumental | tipo de voz]
```

Ejemplo:
```
organic house, tribal downtempo, hand percussion, analog pads, deep bassline, ethnic wooden flute, hypnotic and warm, 112 BPM, instrumental
```

## Reglas duras

- **Sin nombres de artistas reales.** En vez de "como Bonobo", describe: "downtempo organic, jazzy samples, warm vinyl texture, mellow breakbeat".
- **Sin letras ajenas.** Si se pide letra, generar siempre original; nunca reproducir letras existentes.
- **Tags, no prosa.** Separar por comas, evitar frases completas largas.
- **BPM siempre como número** (Suno lo usa como ancla de tempo).
- **Especificar instrumental vs. vocal** explícitamente — si no se indica, Suno a veces mete voz no deseada.

## Mapeo rápido género → BPM típico

| Género | BPM aprox. |
|---|---|
| Organic house / tribal house | 100-122 |
| Melodic techno | 118-126 |
| Downtempo / chillout | 80-105 |
| Psy organic / minimal psy | 110-125 |
| Synthwave | 80-110 |
| Lo-fi hip hop | 70-90 |
| Reggaetón / regional urbano | 85-96 |
| Rock alternativo / shoegaze | 100-140 |

## Vocabulario útil de mood/textura

cálido, hipnótico, eufórico, oscuro, etéreo, terroso, cósmico, melancólico, ritual, onírico, crudo, cristalino, denso, flotante.

## Vocabulario útil de instrumentación orgánica/étnica

hand percussion, djembe, congas, kalimba, wooden flute, charango, oud, analog pads, modular synth, field recordings, vinyl crackle, live guitar, tribal chant vocals.

---
name: suno-prompt-generator
description: Lee una canción o playlist de Spotify (vía URL) y genera prompts listos para usar en Suno AI (style string + duración/BPM sugerido + idea de letra opcional). Úsala cuando el usuario comparta un link de open.spotify.com (canción o playlist), pida "analiza esta playlist", "genera prompts para Suno", "quiero recrear el vibe de esta canción con IA", o mencione Suno junto con una referencia musical. También úsala si el usuario describe un género/mood y pide directamente un prompt de Suno sin playlist de por medio.
---

# Suno Prompt Generator

Convierte una referencia musical (canción o playlist de Spotify) en uno o más prompts de estilo para Suno AI.

## Flujo

1. **Obtener la fuente musical**
   - Si es un link `open.spotify.com/playlist/...` o `.../track/...`: usa `web_fetch` directo sobre la URL. Spotify devuelve metadata Open Graph + el HTML inicial con varias canciones/artistas visibles (no requiere login). Para playlists largas, con los primeros ~20-30 tracks visibles ya es suficiente muestra — no hace falta scrollear todo.
   - Si el usuario solo da nombres de artistas/canciones sueltas (sin link), trabaja directo con eso.
   - Si el usuario tiene el conector de Spotify conectado y pide explícitamente usarlo (o ya lo usó antes en la conversación), puedes usar `Spotify:search` para enriquecer datos puntuales, pero no es necesario para el flujo normal.

2. **Extraer la "huella sonora"** a partir de artistas/tracks identificados:
   - Género/subgénero (ej. organic house, melodic techno, tribal downtempo, synthwave, shoegaze, regional mexicano, etc.)
   - Tempo aproximado (BPM) — infiere por subgénero si no hay dato exacto.
   - Instrumentación característica (percusión orgánica, sintes analógicos, guitarra, flauta nativa, 808s, etc.)
   - Atmósfera/mood (oscuro, eufórico, introspectivo, psicodélico, nostálgico...)
   - Tratamiento vocal si aplica (instrumental, voz susurrada, coros tribales, voz procesada, etc.)
   - Tema/narrativa si el nombre de la playlist o los títulos de canciones sugieren un hilo conductor (ej. nombre de playlist "viaje psicodélico" → temas cósmicos/naturaleza).

3. **Generar el/los prompt(s) de Suno**. Reglas de formato (ver `references/suno-prompt-format.md` para el detalle):
   - **Nunca pongas nombres de artistas reales en el prompt final** — Suno los ignora o los bloquea, y además es mejor práctica describir el sonido en vez de pedir imitación directa. Traduce "suena como X" a sus rasgos sonoros concretos.
   - El campo de estilo es una lista corta de tags separados por coma, no una oración — ej: `organic house, tribal percussion, warm analog pads, deep rolling bassline, ethnic flute, sunset festival, 112 BPM, instrumental`.
   - Incluye siempre: género(s), 2-4 elementos de instrumentación, mood, y BPM aproximado.
   - Indica si es instrumental o con voz; si tiene voz, especifica el tipo (ej. "breathy female vocals", "deep male chant") sin nombrar al artista de referencia.
   - Mantén el style string conciso (apunta a ~150-200 caracteres; Suno trunca/ignora exceso de ruido).

4. **Entrega 3-5 variaciones** cuando la fuente es una playlist completa (para cubrir distintas facetas del sonido), o 1-2 variaciones cuando es una sola canción. Para cada una da: nombre corto descriptivo, el style string listo para copiar, y opcionalmente 2-3 líneas de idea de letra/tema si el usuario quiere canción cantada (no escribas letra completa con derechos de autor de terceros, solo idea original).

5. Si el usuario quiere guardar el resultado, ofrece crear un artifact/archivo, pero por default entrégalo inline en el chat (son prompts cortos para copiar y pegar).

## Notas

- Esto es para crear música ORIGINAL inspirada en una vibra/género, no para clonar una canción específica nota por nota — deja eso claro si el usuario pide replicar algo demasiado literal.
- Si la playlist mezcla géneros muy distintos, agrupa por sub-clusters en vez de promediar todo en un prompt genérico.
- Carlos toca guitarra/batería y produce en Logic Pro — cuando sea relevante, puedes sugerir qué tan fácil sería tocar/grabar esa instrumentación en vivo encima de lo que genere Suno, pero no lo agregues si no lo pide.

## Reference files

- `references/suno-prompt-format.md` — Formato exacto del style string, reglas duras y mapeo género → BPM

---
name: suno-prompt-generator
description: Lee una canción o playlist de Spotify (vía URL) y genera prompts listos para usar en Suno AI (style string + duración/BPM sugerido + idea de letra opcional). Úsala cuando el usuario comparta un link de open.spotify.com (canción o playlist), pida "analiza esta playlist", "genera prompts para Suno", "quiero recrear el vibe de esta canción con IA", o mencione Suno junto con una referencia musical. También úsala si el usuario describe un género/mood y pide directamente un prompt de Suno sin playlist de por medio.
---

# Suno Prompt Generator

Convierte una referencia musical (canción o playlist de Spotify) en uno o más prompts de estilo para Suno AI.

## Flujo

1. **Obtener la fuente musical**
   - Si es un link `open.spotify.com/playlist/...` o `.../track/...`: usa `web_fetch` directo sobre la URL como método principal. Spotify devuelve metadata Open Graph + el HTML inicial con varias canciones/artistas visibles (no requiere login). Para playlists largas, con los primeros ~20-30 tracks visibles ya es suficiente muestra — no hace falta scrollear todo. **No existe una herramienta MCP para traer una playlist completa por ID**, así que `web_fetch` sigue siendo el camino principal para leer una playlist/track específico por URL.
   - Si el usuario solo da nombres de artistas/canciones sueltas (sin link), o si quieres enriquecer/confirmar género y artistas relacionados, usa el conector de **Spotify MCP** (`Spotify:search`, y si aplica `Spotify:get_currently_playing` cuando el usuario pregunta qué está sonando ahora). Está disponible como tool ya conectado — no hace falta `search_mcp_registry` ni `suggest_connectors` para usarlo.
   - Nota: el MCP de Spotify **no expone BPM ni audio-features** (Spotify retiró esa API pública a finales de 2024) ni una llamada para descargar una playlist entera — solo búsqueda, currently-playing y creación de playlists. El tempo se sigue infiriendo por subgénero (ver tabla en `references/suno-prompt-format.md`).
   - **Suno no tiene MCP.** No existe (ni en el directorio de conectores de Claude ni de forma oficial pública conocida) una integración MCP de Suno al momento de escribir esto. Por lo tanto este skill solo *genera el texto del prompt* — el usuario debe pegarlo manualmente en Suno. Si en el futuro aparece un MCP de Suno, actualizar este paso para generar la canción directo.

2. **Extraer la "huella sonora"** a partir de artistas/tracks identificados:
   - Género/subgénero (ej. organic house, melodic techno, tribal downtempo, synthwave, shoegaze, regional mexicano, etc.)
   - Tempo aproximado (BPM) — infiere por subgénero si no hay dato exacto.
   - Instrumentación característica (percusión orgánica, sintes analógicos, guitarra, flauta nativa, 808s, etc.)
   - Atmósfera/mood (oscuro, eufórico, introspectivo, psicodélico, nostálgico...)
   - Tratamiento vocal si aplica (instrumental, voz susurrada, coros tribales, voz procesada, etc.)
   - Tema/narrativa si el nombre de la playlist o los títulos de canciones sugieren un hilo conductor (ej. nombre de playlist "viaje psicodélico" → temas cósmicos/naturaleza).

3. **Generar el/los prompt(s) de Suno**. Reglas de formato (ver `references/suno-prompt-format.md` para el detalle):
   - **Nunca pongas nombres de artistas reales en el prompt final** — Suno los ignora o los bloquea, y además es mejor práctica describir el sonido en vez de pedir imitación directa. Traduce "suena como X" a sus rasgos sonoros concretos. Esto aplica tanto al style string como a la letra: no menciones artistas, canciones existentes, ni marcas registradas.
   - El campo de estilo es una lista corta de tags separados por coma, no una oración — ej: `organic house, tribal percussion, warm analog pads, deep rolling bassline, ethnic flute, sunset festival, 112 BPM, instrumental`.
   - Incluye siempre: género(s), 2-4 elementos de instrumentación, mood, y BPM aproximado.
   - Indica si es instrumental o con voz; si tiene voz, especifica el tipo (ej. "breathy female vocals", "deep male chant") sin nombrar al artista de referencia.
   - Mantén el style string conciso (apunta a ~150-200 caracteres; Suno trunca/ignora exceso de ruido).

4. **Entrega 3-5 variaciones** cuando la fuente es una playlist completa (para cubrir distintas facetas del sonido), o 1-2 variaciones cuando es una sola canción. Si el usuario pide algo rápido o se nota que quiere algo casual, puedes dar solo 1-2 variaciones aunque sea playlist — usa criterio. Para cada una da: nombre corto descriptivo, el style string listo para copiar, y opcionalmente 2-3 líneas de idea de letra/tema si el usuario quiere canción cantada (no escribas letra completa con derechos de autor de terceros, solo idea original).

5. Si el usuario quiere guardar el resultado, ofrece crear un artifact/archivo, pero por default entrégalo inline en el chat (son prompts cortos para copiar y pegar).

6. **Flujo conversacional**: si el usuario comparte un link sin contexto adicional ("mira esto", "qué te parece esta playlist"), no te limites a listar canciones. Primero procesa la playlist con `web_fetch`, luego saca conclusiones sobre el género/mood colectivo y genera prompts directo. Si el usuario pide ajustes sobre los prompts generados (cambiar BPM, hacerlo más oscuro, agregar guitarra, etc.), itera sobre la misma playlist ya analizada sin volver a pedir la URL.

## Notas

- Esto es para crear música ORIGINAL inspirada en una vibra/género, no para clonar una canción específica nota por nota — deja eso claro si el usuario pide replicar algo demasiado literal.
- Si la playlist mezcla géneros muy distintos, agrupa por sub-clusters en vez de promediar todo en un prompt genérico.
- Carlos toca guitarra/batería y produce en Logic Pro — cuando sea relevante, puedes sugerir qué tan fácil sería tocar/grabar esa instrumentación en vivo encima de lo que genere Suno, pero no lo agregues si no lo pide.

## Reference files

- `references/suno-prompt-format.md` — Formato exacto del style string, reglas duras y mapeo género → BPM

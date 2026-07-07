# export a single file
pocket-tts export-voice voice_memo127762.mp3 jack.safetensors

# export an online file to current directory
pocket-tts export-voice https://huggingface.co/kyutai/tts-voices/resolve/main/alba-mackenna/announcer.wav ./announcer.safetensors

# use the exported safetensors
pocket-tts generate --text "Hello, welcome to today's game between the Bears and Cubs."  --voice announcer.safetensors


# IMPORTANT: export-voice and generate MUST use the same --language model.
# The voice state stores KV caches keyed by transformer layer names.
# A 6-layer model (english) produces layers 0-5; a 24-layer model (spanish_24l) produces 0-23.
# Mismatching them causes KeyError: 'transformer.layers.6.self_attn' at generation time.

uv run pocket-tts export-voice voices\Iosux.wav export\iosu.safetensors --language spanish_24l

uv run pocket-tts generate --text "hola como estas" --voice export\iosu.safetensors --language spanish_24l

# ---------------------------------------------------------------------------
# wslc (WSL container) — construir y ejecutar con Docker en WSL
# ---------------------------------------------------------------------------

# Construir la imagen (el contexto es el DIRECTORIO, no el Dockerfile)
wslc build -t pocket-tts c:\IA\voice\VOICE-pocket-tts

# Ver la ayuda del CLI dentro del contenedor
wslc run pocket-tts --help

# Servidor web — expone el API en http://localhost:8069
wslc run -p 8069:8000 pocket-tts serve --host 0.0.0.0

# Exportar voz dentro del contenedor (monta export/ y voices/ para leer/escribir)
wslc run -v c:\IA\voice\VOICE-pocket-tts\export:/export -v c:\IA\voice\VOICE-pocket-tts\voices:/voices pocket-tts export-voice /voices/Iosux.wav /export/iosu.safetensors --language spanish_24l

# Generar audio dentro del contenedor (monta export/ para leer la voz y escribir el wav)
wslc run -v c:\IA\voice\VOICE-pocket-tts\export:/export pocket-tts generate --text "Perfecto, Eita. Me alegra mucho que ya funcione el audio. Vete a descansar. Gracias por la paciencia hoy." --voice /export/iosu.safetensors --language spanish_24l --output-path /export/salida.wav

# Conservar la caché de pesos entre ejecuciones (evita re-descargar de HuggingFace)
wslc run -v pocket_tts_cache:/root/.cache -p 8069:8000 pocket-tts serve --host 0.0.0.0

# docker-compose — levanta el servidor con caché y puertos ya configurados
wslc compose up

# NOTA: el ENTRYPOINT del Dockerfile es "uv run pocket-tts",
# así que solo se pasa el subcomando (serve, generate, export-voice, --help).

# ---------------------------------------------------------------------------
# Ejemplo completo: servidor + petición HTTP con wslc
# ---------------------------------------------------------------------------

# 1) Arrancar el servidor en wslc con el modelo spanish_24l y caché persistente.
#    -p 8069:8000  →  expone el API en http://localhost:8069
#    -v ...cache   →  conserva los pesos descargados de HuggingFace entre reinicios
#    --env-file    →  lee HF_TOKEN del .env del proyecto (token de HuggingFace para voice cloning)
#
#    IMPORTANTE: voice cloning requiere el modelo gated de https://huggingface.co/kyutai/pocket-tts
#    1. Acepta los términos en esa página con tu cuenta de HuggingFace.
#    2. Crea un token Read en https://huggingface.co/settings/tokens
#    3. Guárdalo en .env del proyecto:  HF_TOKEN=hf_tu_token
wslc run --env-file c:\IA\voice\VOICE-pocket-tts\.env -v pocket_tts_cache:/root/.cache -p 8069:8000 pocket-tts serve --host 0.0.0.0 --language spanish_24l

# 2) (en otra terminal) Generar audio llamando al API con la voz personalizada.
#    El endpoint POST /tts acepta: text, voice_url o voice_wav (archivo .wav).
#    Aquí subimos voices\Iosux.wav como voice_wav y guardamos la respuesta en export\salida.wav.
curl -s -X POST http://localhost:8069/tts -F "text=Perfecto, Eita. Me alegra mucho que ya funcione el audio. Vete a descansar. Gracias por la paciencia hoy." -F "voice_wav=@c:\IA\voice\VOICE-pocket-tts\voices\Iosux.wav" -o c:\IA\voice\VOICE-pocket-tts\export\salida.wav

# 3) Sin voz personalizada (usa la voz por defecto del idioma, 'lola' para español):
curl -s -X POST http://localhost:8069/tts -F "text=Hola, esto es una prueba sin voz personalizada." -o c:\IA\voice\VOICE-pocket-tts\export\salida_default.wav

# 4) Usando una voz online de HuggingFace (voice_url con hf://):
curl -s -X POST http://localhost:8069/tts -F "text=Hola, esta es la voz announcer de Alba Mackenna." -F "voice_url=hf://kyutai/tts-voices/alba-mackenna/announcer.wav" -o c:\IA\voice\VOICE-pocket-tts\export\salida_alba.wav

# 5) Comprobar que el servidor responde:
curl -s http://localhost:8069/health


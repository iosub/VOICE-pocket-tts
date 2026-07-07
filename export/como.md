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
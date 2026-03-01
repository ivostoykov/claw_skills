# Local Whisper (cpp)

Local speech-to-text using Whisper.cpp. Runs fully offline with downloaded models.

## When to use

Use this skill when the user asks to:
- Transcribe audio files
- Convert speech to text
- Get transcription from audio (mp3, wav, m4a, flac, aac, ogg, wma)

## How to execute

Run the whisper_cpp command with the audio file path:

/PATH/TO/bin/whisper_cpp /path/to/audio.mp3

Optional: Add --model=medium or --language=en if needed.

## Output

Return the raw transcription output without modification.
